<template>
<q-dialog
  v-bind="$attrs"
  ref="dialogRef"
  @hide="onDialogHide"
>
  <q-card class="q-dialog-plugin">
    <q-form ref="form" @submit="onOKClick">
      <q-card-section class="row items-center no-wrap q-pa-xs bg-secondary text-white">
        <div class="text-weight-medium">
          {{ tipoExibicao === 'adicionar' ? 'Nova' : 'Editar' }} Tarefa
        </div>
        <q-space />
        <q-btn
          icon="close"
          dense
          round
          flat
          v-close-popup
        />
      </q-card-section>

      <q-card-section class="q-mt-md">
          <q-input
            v-model="tarefa"
            label="Nome da tarefa"
            type="textarea"
            :rules="[val => !!val || 'Campo obrigatório']"
            dense
            filled
            autofocus
          />

      </q-card-section>

      <q-card-actions align="right">
        <q-btn
          label="Cancelar"
          color="primary"
          no-caps
          flat
          @click="onDialogCancel"
        />
        <q-btn
          :label="tipoExibicao === 'adicionar' ? 'Adicionar' : 'Editar'"
          color="primary"
          type="submit"
          no-caps
        />
      </q-card-actions>

    </q-form>
  </q-card>
</q-dialog>
</template>

<script>
import { useDialogPluginComponent } from 'quasar'

export default {
  name: 'ModalForm',

  props: {
    tipoExibicao: {
      type: String,
      required: false,
      default: 'adicionar'
    },
    descricao: {
      type: String,
      required: false,
      default: ''
    }
  },

  setup () {
    const {
      dialogRef,
      onDialogHide,
      onDialogOK,
      onDialogCancel
    } = useDialogPluginComponent()

    return {
      dialogRef,
      onDialogHide,
      onDialogOK,
      onDialogCancel,
    }
  },

  emits: [...useDialogPluginComponent.emits],

  data () {
    return {
      tarefa: this.tipoExibicao === 'adicionar' ? '' : this.descricao
    }
  },

  methods: {
    async onOKClick () {
      const formValido = await this.$refs.form.validate()

      if (!formValido) return

      this.onDialogOK({ tarefa: this.tarefa })
    }
  }
}
</script>
