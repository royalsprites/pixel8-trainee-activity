<template>
  <q-page class="q-pa-md">
    <q-card class="q-pa-md shadow-2 rounded-borders">
      <q-card-section>
        <div class="text-h5 text-primary">Finished Tasks</div>
      </q-card-section>

      <q-separator />

      <q-card-section>
        <q-list bordered separator v-if="finishedTasks.length > 0">
          <q-item v-for="task in finishedTasks" :key="task.id">
            <q-item-section>
              <div class="text-strike text-grey">{{ task.title }}</div>
            </q-item-section>
            <q-item-section side>
              <q-btn
                flat
                icon="delete"
                color="negative"
                @click="removeFinishedTask(task.id)"
              />
            </q-item-section>
          </q-item>
        </q-list>

        <div v-else class="text-grey text-center q-py-lg">
          No finished tasks yet
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const finishedTasks = ref([])

onMounted(() => {
  fetchFinishedTasks()
})

const fetchFinishedTasks = () => {
  const savedFinishedTasks = localStorage.getItem('finishedTasks')
  finishedTasks.value = savedFinishedTasks ? JSON.parse(savedFinishedTasks) : []
}

const removeFinishedTask = (id) => {
  finishedTasks.value = finishedTasks.value.filter(task => task.id !== id)
  localStorage.setItem('finishedTasks', JSON.stringify(finishedTasks.value))
}
</script>
