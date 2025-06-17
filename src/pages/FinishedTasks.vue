<template>
  <q-page class="q-pa-md q-gutter-md">
    <q-card class="q-pa-md shadow-2 rounded-borders">
      <q-card-section class="q-px-none">
        <div class="text-h5 text-primary">Finished Tasks</div>
      </q-card-section>

      <q-separator />

      <q-card-section>
        <div v-if="finishedTasks.length === 0" class="text-subtitle1 text-grey-7">
          No finished tasks yet. Get to work!
        </div>

        <q-list v-else bordered separator>
          <q-item v-for="task in finishedTasks" :key="task.id">
            <q-item-section>
              <div class="text-grey-7">
                <q-icon name="check_circle" color="positive" class="q-mr-sm" />
                {{ task.title }}
              </div>
            </q-item-section>
            <q-item-section side>
              <q-btn
                icon="delete"
                color="negative"
                flat
                round
                dense
                @click="confirmDelete(task.id)"
              >
                <q-tooltip>Delete</q-tooltip>
              </q-btn>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const finishedTasks = ref([])

onMounted(() => {
  const saved = localStorage.getItem('finishedTasks')
  if (saved) {
    try {
      finishedTasks.value = JSON.parse(saved)
    } catch (e) {
      console.error('Failed to load finished tasks:', e)
    }
  }
})

const confirmDelete = (id) => {
  finishedTasks.value = finishedTasks.value.filter(t => t.id !== id)
  localStorage.setItem('finishedTasks', JSON.stringify(finishedTasks.value))
}
</script>
