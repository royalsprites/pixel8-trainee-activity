<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title>
          To-Do App
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>
        <q-item-label header>
          Essential Links
        </q-item-label>

        <NavItem
          v-for="link in navItems"
          :key="link.label"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import NavItem from 'components/NavItem.vue'

const leftDrawerOpen = ref(false)

const navItems = [
  {
    label: 'Home',
    icon: 'home',
    to: '/'
  },
  {
    label: 'Finished Tasks',
    icon: 'done_all',
    to: '/finished'
  },
  {
    label: 'About',
    icon: 'info',
    to: '/about'
  }
]

function toggleLeftDrawer () {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>
