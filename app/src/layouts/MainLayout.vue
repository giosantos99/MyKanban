<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title>
          <q-avatar>
            <img src="@/assets/logo.png" />
          </q-avatar>
            MyKanban
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container class="page-container">
      <router-view />
    </q-page-container>

  </q-layout>
</template>

<style lang="sass" scoped>
.page-container
  max-width: 1920px
  margin: 0 auto
  background: $accent
</style>
