<template>
<q-dialog ref="dialogRef" @hide="onDialogHide">
  <q-card class="q-dialog-plugin">
    <q-card-section class="row items-center bg-secondary text-white q-pa-xs">
      <div class="text-weight-medium">{{ titulo }}</div>
      <q-space />
      <q-btn
        icon="close"
        v-close-popup
        round
        dense
        flat
      />
    </q-card-section>

    <q-card-section class="flex flex-center q-pa-sm q-mt-md">
      <p>{{ texto }}</p>
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
        label="Excluir"
        color="primary"
        no-caps
        @click="onDialogOK"
      />
    </q-card-actions>
  </q-card>
</q-dialog>
</template>

<script>
import { useDialogPluginComponent } from 'quasar'

export default {
  name: 'ModalNotificacao',

  props: {
    titulo: {
      type: String,
      required: true
    },
    texto: {
      type: String,
      required: true
    }
  },

  emits: [...useDialogPluginComponent.emits],

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
  }
}
</script>
