<template>
  <q-layout view="lHh Lpr lFf">
    <Sidebar />

    <q-page-container class="">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import Sidebar from 'components/SideBar.vue';
</script>
