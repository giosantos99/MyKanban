<template>
<q-card class="card">
  <q-card-section
    class="row items-center full-height q-pa-md"
    style="min-height: 100px"
  >
    <div class="text-body ellipsis-2-lines text-subtitle1">{{ tarefa.descricao || '' }}</div>
  </q-card-section>

  <q-card-section class="row items-center q-pa-xs q-pt-md bg-grey-2">
    <div class="row items-center">
      <q-btn
        v-if="$q.screen.lt.md && (exibirSetaCima || exibirSetaBaixo)"
        icon="fa-solid fa-up-down"
        color="primary"
        size="sm"
        round
        dense
        flat
      >
        <q-menu>
          <q-list
            v-for="item in menu_setas"
            :key="item.label"
          >
            <q-item
              v-if="item.exibir"
              clickable
              v-ripple
              v-close-popup
              @click="item.action(item.icon)"
            >
              <q-item-section avatar>
                <q-icon color="secondary" :name="item.icon" />
              </q-item-section>

              <q-item-section>{{ item.label }}</q-item-section>
            </q-item>
          </q-list>
        </q-menu>
      </q-btn>
      <span :class="[
          `${$q.screen.lt.md ? 'q-pl-sm' : ''}`,
          'text-caption'
        ]"
      >
        {{ tarefa.dt_criacao || ''}}
      </span>

    </div>
    <q-space />
    <q-btn
      icon="settings"
      color="primary"
      dense
      round
      flat
    >
      <q-menu>
        <q-list
          v-for="config in configuracoes"
          :key="config.label"
        >
          <q-item
            v-if="config.label !== 'Mover' ? true : $q.screen.lt.md"
            clickable
            v-ripple
            v-close-popup
            @click="config.action"
          >
            <q-item-section avatar>
              <q-icon color="secondary" :name="config.icon" />
            </q-item-section>

            <q-item-section>{{ config.label }}</q-item-section>
          </q-item>
        </q-list>
      </q-menu>

      <q-tooltip v-if="!$q.screen.lt.md">Editar tarefa</q-tooltip>

    </q-btn>
  </q-card-section>
</q-card>
</template>

<script>
import { defineAsyncComponent } from 'vue'

export default {
  name: 'CardTarefa',

  props: {
    tarefa: {
      type: Object,
      required: true
    },
    exibirSetaCima: {
      type: Boolean,
      required: false
    },
    exibirSetaBaixo: {
      type: Boolean,
      required: false
    }
  },

  emits: [
    'alterar-painel',
    'editar-tarefa',
    'excluir-tarefa',
    'mover-tarefa'
  ],

  data () {
    return {
      configuracoes: [
        {
          label: 'Editar',
          icon: 'edit',
          action: this.editarTarefa
        },
        {
          label: 'Excluir',
          icon: 'delete',
          action: this.excluirTarefa
        },
        {
          label: 'Alterar Painel',
          icon: 'fa-solid fa-table-columns',
          action: this.alterarPainel
        },
      ],
      menu_setas: [
        {
          label: 'Mover para cima',
          icon: 'arrow_drop_up',
          exibir: this.exibirSetaCima,
          action: this.moverTarefa
        },
        {
          label: 'Mover para baixo',
          icon: 'arrow_drop_down',
          exibir: this.exibirSetaBaixo,
          action: this.moverTarefa
        }
      ]
    }
  },

  methods: {
    editarTarefa () {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(() => import('@/components/Modal/ModalForm.vue')),
        componentProps: {
          tipoExibicao: 'editar',
          descricao: this.tarefa.descricao
        }
      })

      dialog.onOk(({ tarefa: descricao }) => this.$emit('editar-tarefa', {
          ...this.tarefa,
          descricao
        }
      ))
    },

    excluirTarefa () {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(() => import('@/components/Modal/ModalNotificacao.vue')),
        componentProps: {
          texto: `Tem certeza que deseja excluir a tarefa "${this.tarefa.descricao}" ?`,
          titulo: 'Excluir Tarefa'
        }
      })

      dialog.onOk(() => this.$emit('excluir-tarefa', this.tarefa.id))
    },

    alterarPainel () {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(() => import('@/components/Modal/ModalMoverTarefa.vue')),
        componentProps: {
          status: this.tarefa.status
        }
      })

      dialog.onOk(({ novoPainel }) => this.$emit('alterar-painel', novoPainel))
    },

    moverTarefa (action) {
      this.$emit('mover-tarefa', action)
    }
  }
}
</script>

<style lang="sass" scoped>
.card
  background-color: #ffffff
  border-radius: 8px
  margin-bottom: 12px


  border-left: 5px solid
  border-image: linear-gradient(180deg, $secondary 0%, $primary 100%) 1

  display: flex
  flex-direction: column
  gap: 12px
  transition: transform 0.2s ease, box-shadow 0.2s ease

  &:hover
    transform: translateY(-2px)
    box-shadow: 0 6px 12px -2px rgba(0, 0, 0, 0.12)

</style>
