<template>
<q-page v-if="!loading" class="q-pa-sm">

  <div class="container row no-wrap scroll">
    <template v-if="paineis?.length">
      <div
        v-for="painel in paineis"
        :key="painel.id"
        :style="{ order: painel.ordenacao_painel }"
        :class="[
          'col-xs-12 col-md-4 q-px-sm text-white q-pb-sm',
          `painel-${painel.id}`
        ]"
      >
        <q-card class="fit painel bg-accent">
          <q-card-section
            class="row justify-between items-center q-pa-xs bg-secondary"
            style="min-height: 42px"
          >

            <div class="row items-center no-wrap">
              <q-icon :name="painel.icone" class="q-mr-sm" />
              <span class="text-weight-medium">{{ painel.status }}</span>
            </div>

            <q-btn
              v-if="tarefas[painel.id]?.length"
              icon="delete_sweep"
              round
              dense
              flat
              @click="excluirTarefasDoPainel(painel)"
            >
              <q-tooltip v-if="!$q.screen.lt.md">Limpar tudo</q-tooltip>
            </q-btn>
          </q-card-section>

          <div class="container-painel column no-wrap scroll text-dark">

            <div
              v-if="!tarefas[painel.id]?.length"
              class="flex flex-center items-center q-gutter-x-sm text-grey-7 q-pt-xl"
            >
              <q-icon name="warning" size="sm" color="warning" />
              <span>Nenhuma tarefa adicionada.</span>
            </div>

            <LayoutCardsWeb
              v-if="!$q.screen.lt.md"

              :_tarefas="tarefas"
              :painelId="painel.id"

              @excluir-tarefa="idTarefa => excluirTarefa(painel.id, idTarefa)"
              @editar-tarefa="tarefa => editarTarefa(painel.id, tarefa)"
              @drag-drop="({ evt, tarefas }) => onDragDrop(evt, tarefas, painel.id)"
            />

            <!-- LAYOUT MOBILE -->
            <CardTarefa
              v-else
              v-for="(tarefa, index) in tarefas[painel.id]"
              :key="tarefa.id"

              :tarefa="tarefa"
              :exibirSetaCima="!!index"
              :exibirSetaBaixo="index !== tarefas[painel.id].length - 1"

              :style="{ order: tarefa.ordenacao_tarefa }"
              class="q-ma-md"

              @alterar-painel="novoPainel => alterarPainelMobile(novoPainel, painel.id, tarefa)"
              @editar-tarefa="tarefa => editarTarefa(painel.id, tarefa)"
              @excluir-tarefa="idTarefa => excluirTarefa(painel.id, idTarefa)"
              @mover-tarefa="action => moverTarefaMobile(action, painel.id, tarefa, index)"
            />
          </div>
        </q-card>
      </div>
    </template>
  </div>

  <img
    v-if="mostrarComemoracao"
    src="@/assets/done.gif"
    class="done"
  />

  <ButtonNovaTarefa @adicionar-tarefa="modalAdicionarTarefa"/>

</q-page>

<q-page v-else class="q-pa-sm">
  <div class="container row no-wrap scroll">
    <template v-for="n in 3" :key="n">
      <div class="col-xs-12 col-md-4 q-px-sm text-white q-pb-sm">
        <q-skeleton square class="fit painel" />
      </div>
    </template>
  </div>

</q-page>
</template>

<script>
import { defineAsyncComponent } from 'vue'

import { ButtonNovaTarefa, CardTarefa } from '@/components/UI'
import LayoutCardsWeb from '@/components/LayoutCardsWeb.vue'

export default {
  components: {
    ButtonNovaTarefa,
    CardTarefa,
    LayoutCardsWeb
  },

  data () {
    return {
      loading: false,
      mostrarComemoracao: false,
      paineis: [
        {
          status: 'Fazer',
          id: 'fazer',
          icone: 'play_circle',
          ordenacao_painel: 1
        },
        {
          status: 'Fazendo',
          id: 'fazendo',
          icone: 'fa-solid fa-spinner',
          ordenacao_painel: 2
        },
        {
          status: 'Feito',
          id: 'feito',
          icone: 'check_circle',
          ordenacao_painel: 3
        }
      ],
      tarefas: {}
    }
  },

  methods: {
    async moverTarefaMobile (action, painel, tarefa, index) {
      const from = action === 'arrow_drop_up' ? index - 1 : index + 1

      const tarefas = this.tarefas[painel]

      tarefas.splice(index, 0, tarefas.splice(from, 1)[0])

      this.tarefas[painel] = tarefas.map((tarefa, ordenacao_tarefa) => ({
        ...tarefa,
        ordenacao_tarefa
      }))

      await this.onDragDrop(
        { added: undefined, moved: { element: tarefa } },
        this.tarefas,
        painel
      )

    },

    async alterarPainelMobile (novoPainel, painelAtual, tarefa) {

      this.tarefas[novoPainel].push(tarefa)
      this.tarefas[painelAtual] = this.tarefas[painelAtual]?.filter(({ id }) => id !== tarefa.id)

      await this.onDragDrop(
        { moved: undefined, added: { element: { ...tarefa }} },
        this.tarefas,
        novoPainel
      )
      await this.onDragDrop(
        { moved: undefined, added: undefined },
        this.tarefas,
        painelAtual
      )

      const containerPainel = document.querySelector(`.painel-${novoPainel}`)

      containerPainel?.scrollIntoView({
        behavior: 'smooth'
      })

    },

    async onDragDrop ({ moved, added }, _tarefas, status) {
      try {
        this.tarefas = _tarefas

        const elAdicionado = added?.element
        const elMovido = moved?.element

        if (elMovido) {
          this.tarefas[status] = this.tarefas[status].map(
            (tarefa, index) => ({
              ...tarefa,
              ordenacao_tarefa: index
            })
          )

          await Promise.all(
            this.tarefas[status].map(tarefa =>
              this.$api.put(
                `${status}/${tarefa.id}`,
                tarefa
              )
            )
          )
        }

        if (elAdicionado) {

          const statusAnterior = elAdicionado.status

          const { data: novaTarefa } = await this.fnAdicionarTarefa(
            status,
            elAdicionado.descricao,
            false
          )

          await this.excluirTarefa(
            statusAnterior,
            elAdicionado.id,
            false
          )

          const index = this.tarefas[status].findIndex(
            tarefa => tarefa.id === elAdicionado.id
          )

          if (index !== -1) this.tarefas[status].splice(index, 1, novaTarefa)

          this.tarefas[status] = this.tarefas[status].map(
            (tarefa, index) => ({
              ...tarefa,
              status,
              ordenacao_tarefa: index
            })
          )

          await Promise.all(
            this.tarefas[status].map(tarefa =>
              this.$api.put(
                `${status}/${tarefa.id}`,
                tarefa
              )
            )
          )

          this.tarefas[statusAnterior] =
            this.tarefas[statusAnterior].map(
              (tarefa, index) => ({
                ...tarefa,
                ordenacao_tarefa: index
              })
            )

          await Promise.all(
            this.tarefas[statusAnterior].map(tarefa =>
              this.$api.put(
                `${statusAnterior}/${tarefa.id}`,
                tarefa
              )
            )
          )

        const { fazer, fazendo, feito } = this.tarefas

        if ( !fazer.length && !fazendo.length && feito.length > 0 ) {
          this.mostrarComemoracao = true

          setTimeout(() => this.mostrarComemoracao = false, 1600)
        }
      }

      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Ocorreu um erro ao mudar a posição da tarefa'
        })

        await this.fetchTarefas()
      }
    },

    excluirTarefasDoPainel ({ status, id }) {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(() => import('@/components/Modal/ModalNotificacao.vue')),
        componentProps: {
          texto: `Tem certeza que deseja excluir todas as tarefas do painel "${status}" ?`,
          titulo: 'Excluir Tarefas'
        }
      })

      dialog.onOk(async () => {
        try {
          const ids = this.tarefas[id]
            ?.map(({ id }) => id)
            ?? []

          if (!ids.length) return

          await Promise.all(ids
            .map(idTarefa => this.$api.delete(`${id}/${idTarefa}`))
          )

          await this.fetchTarefas()

          this.$q.notify({
            color: 'positive',
            message: `${ids.length} tarefa(s) excluída(s) com sucesso`
          })
        } catch {
          this.$q.notify({
            color: 'negative',
            message: 'Erro ao excluir as tarefas do painel'
          })
        }
      })
    },

    async editarTarefa (painel, tarefa) {
      try {
        const dt_criacao = new Intl.DateTimeFormat('pt-Br', { dateStyle: 'short' }).format(new Date())

        await this.$api.put(`${painel}/${tarefa.id}`, {
          ...tarefa,
          dt_criacao
        })

        this.fetchTarefas()

        this.$q.notify({
          color: 'positive',
          message: 'Tarefa editada com sucesso'
        })
      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Erro ao editar a tarefa'
        })
      }
    },

    async excluirTarefa (painel, id, notificacao = true) {
      try {
        await this.$api.delete(`${painel}/${id}`)

        notificacao && this.fetchTarefas()

        notificacao && this.$q.notify({
          color: 'positive',
          message: 'Tarefa excluída com sucesso'
        })
      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Erro ao excluir a tarefa'
        })
      }
    },

    async fnAdicionarTarefa (status, descricao, notificacao = true) {
      try {
        const ordenacao_tarefa = this.tarefas[status]?.length
          ? Math.max(...this.tarefas[status].map(({ ordenacao_tarefa }) => ordenacao_tarefa)) + 1
          : 0

        const dt_criacao = new Intl.DateTimeFormat('pt-Br', {
            dateStyle: 'short'
          }).format(new Date())

        const data = await this.$api.post(status, {
          status,
          descricao,
          ordenacao_tarefa,
          dt_criacao
        })

        notificacao && this.fetchTarefas()

        notificacao &&  this.$q.notify({
          color: 'positive',
          message: 'Tarefa adicionada com sucesso'
        })

        return data
      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Ocorreu um erro ao adicionar a tarefa'
        })
      }
    },

    modalAdicionarTarefa () {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(() => import('@/components/Modal/ModalForm.vue')),
      })

      dialog.onOk(({ tarefa }) => {
        this.fnAdicionarTarefa('fazer', tarefa)

        if (this.$q.screen.lt.md) {
          const painelFazer = document.querySelector('.painel-fazer')

          painelFazer?.scrollIntoView({
            behavior: 'smooth'
          })
        }
      })
    },

    async fetchTarefas () {
      try {
        this.loading = true

        const { data: fazer } = await this.$api.get('fazer?_sort=ordenacao_tarefa')
        const { data: fazendo } = await this.$api.get('fazendo?_sort=ordenacao_tarefa')
        const { data: feito } = await this.$api.get('feito?_sort=ordenacao_tarefa')

        this.tarefas = {
          fazer,
          fazendo,
          feito
        }

      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Erro ao buscar os dados das tarefas'
        })
      } finally {
        this.loading = false
      }
    }
  },

  mounted () {
    this.fetchTarefas()
  }
}
</script>

<style lang="sass" scoped>

.container
  min-height: calc( 100vh - 66px )
  max-height: calc( 100vh - 66px )

  &-painel
    max-height: calc( 100vh - 120px )

.done
  position: fixed
  top: 50%
  left: 50%
  transform: translate(-50%, -50%)
  z-index: 999
  pointer-events: none
  max-width: 300px
  max-height: 300px

</style>
