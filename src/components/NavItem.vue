<template>
  <q-item clickable :to="props.to" tag="router-link">
    <q-item-section avatar v-if="props.icon">
      <q-icon :name="props.icon" />
    </q-item-section>

    <q-item-section>
      <q-item-label>{{ props.label }}</q-item-label>
    </q-item-section>
  </q-item>
</template>

<script setup>
const props = defineProps({
  label: {
    type: String,
    required: true
  },
  to: {
    type: String,
    required: true
  },
  icon: {
    type: String,
    default: ''
  }
})
</script>
