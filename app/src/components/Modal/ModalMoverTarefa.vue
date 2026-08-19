<template>
<q-dialog ref="dialogRef" @hide="onDialogHide">
  <q-card class="q-dialog-plugin">
    <q-form ref="form" @submit="onOKClick">
      <q-card-section class="row items-center bg-secondary text-white q-pa-xs">
        <div class="text-weight-medium">Alterar Painel</div>
        <q-space />
        <q-btn
          icon="close"
          v-close-popup
          round
          dense
          flat
        />
      </q-card-section>

      <q-card-section class="flex flex-center q-pa-sm">
        <p class="q-pt-lg">Selecione a coluna para a qual deseja mover esta tarefa:</p>

        <q-field
          v-model="painel_selecionado"
          :rules="[val => !!val || 'Selecione uma das opções']"
          borderless
          class="campo-validacao"
        >
          <template #control>
            <div class="row items-center justify-center full-width">
              <q-radio
                v-model="painel_selecionado"
                v-for="coluna in colunas.filter(coluna => coluna.value !== status)"
                :key="coluna.value"
                :val="coluna.value"
                :label="coluna.label"
              />
            </div>
          </template>
        </q-field>

      </q-card-section>

      <q-card-actions align="right">
        <q-btn
          label="Cancelar"
          color="primary"
          no-caps
          flat
          @click="onCancelClick"
        />
        <q-btn
          label="Mover"
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
  name: 'ModalMoverTarefa',

  emits: [...useDialogPluginComponent.emits],

  props: {
    status: {
      type: String,
      required: true
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
      onCancelClick: onDialogCancel
    }
  },

  data () {
    return {
      painel_selecionado: '',
      colunas: [
        { label: 'Fazer', value: 'fazer' },
        { label: 'Fazendo', value: 'fazendo' },
        { label: 'Feito', value: 'feito' }
      ]
    }
  },

  methods: {
    async onOKClick () {
      const formValido = await this.$refs.form.validate()

      if (!formValido) return

      this.onDialogOK({ novoPainel: this.painel_selecionado })
    }
  }
}
</script>

<style lang="sass" scoped>
.campo-validacao
  padding: 0
  margin: 0
  min-height: 0

  :deep(.q-field__control)
    min-height: 0
    padding: 0

  :deep(.q-field__bottom)
    padding: 0
</style>
