<template>
  <q-page-sticky position="bottom-right" :offset="[18, 18]">
    <q-btn
      icon="add"
      color="primary"
      padding="sm"
      dense
      fab
      @click="$emit('adicionar-tarefa')"
    >
      <q-tooltip
        v-if="!$q.screen.lt.md"
        anchor="center right"
        self="center left"
        :offset="[10, 10]"
      >
        Adicionar tarefa
      </q-tooltip>
    </q-btn>
  </q-page-sticky>
</template>

<script>
export default {
  name: 'AddNovaTarefa',

  emits: ['adicionar-tarefa']
}
</script>
