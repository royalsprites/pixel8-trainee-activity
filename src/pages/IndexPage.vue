<template>
  <q-page class="q-pa-md q-gutter-md">
    <q-card class="q-pa-md shadow-2 rounded-borders">
      <q-card-section class="q-px-none">
        <div class="text-h5 text-primary">Tasks</div>
      </q-card-section>

      <div class="row q-gutter-md q-mb-md">
        <q-input
          v-model="newTaskName"
          label="Anything to do?"
          outlined
          class="col"
          @keyup.enter="addTask"
        />
        <q-btn
          icon="add"
          color="primary"
          @click="addTask"
          :disable="!newTaskName.trim()"
        >
          <span class="q-mr-xs">Add Task</span>
        </q-btn>
      </div>

      <q-separator />

      <q-card-section>
        <div class="text-subtitle1 text-weight-medium q-mb-sm">Your Tasks</div>
        <div
          v-for="task in tasks"
          :key="task.id"
          class="q-pa-sm q-my-sm bg-grey-2 rounded-borders shadow-1"
        >
          <div class="row items-center q-gutter-sm">
            <q-input
              v-model="task.title"
              dense
              outlined
              class="col-grow q-mb-none q-pb-none"
              @keyup.enter="updateTask(task)"
              @blur="updateTask(task)"
              :rules="[val => !!val.trim() || 'Task title is required']"
            />
            <q-btn
              icon="edit"
              color="secondary"
              flat
              round
              dense
              @click="updateTask(task)"
              :disable="!task.title.trim()"
            >
              <q-tooltip>Update</q-tooltip>
            </q-btn>
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
            <q-btn
              icon="check_circle"
              color="positive"
              flat
              round
              dense
              @click="markTaskAsFinished(task)"
            >
              <q-tooltip>Mark as Finished</q-tooltip>
            </q-btn>
          </div>
        </div>
      </q-card-section>
    </q-card>

    <q-dialog v-model="confirmDeleteDialog">
      <q-card>
        <q-card-section>
          <div class="text-h6">Confirm Delete</div>
        </q-card-section>

        <q-card-section>
          Are you sure you want to delete this task?
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="primary" v-close-popup />
          <q-btn
            flat
            label="Delete"
            color="negative"
            @click="deleteTask(deleteTaskId, deleteFromFinished)"
            v-close-popup
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { v4 as uuidv4 } from 'uuid'
import { api } from 'boot/axios'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const tasks = ref([])
const finishedTasks = ref([])
const newTaskName = ref('')
const confirmDeleteDialog = ref(false)
const deleteTaskId = ref(null)
const deleteFromFinished = ref(false)

onMounted(() => {
  fetchTasks()
  loadFinishedTasks()
  loadLocalTasks()
})

const localTasks = ref([])

const loadLocalTasks = () => {
  const savedLocal = localStorage.getItem('localTasks')
  if (savedLocal) {
    try {
      localTasks.value = JSON.parse(savedLocal)
    } catch (e) {
      console.error('Failed to parse local tasks:', e)
      localTasks.value = []
    }
  }
}

const fetchTasks = async () => {
  try {
    const res = await api.get('https://jsonplaceholder.typicode.com/todos')
    const apiTasks = res.data.filter(task => !task.completed).slice(0, 10)

    const finishedIds = new Set(finishedTasks.value.map(t => t.id))
    const combined = [...localTasks.value, ...apiTasks]

    tasks.value = combined.filter(task => !finishedIds.has(task.id))
  } catch (err) {
    console.error('Failed to fetch tasks:', err)
    $q.notify({
      type: 'negative',
      message: 'Failed to load tasks'
    })
    tasks.value = [...localTasks.value]
  }
}

const addTask = async () => {
  const title = newTaskName.value.trim()
  if (!title) return

  const newTask = {
    id: uuidv4(),
    title,
    completed: false
  }

  localTasks.value.unshift(newTask)
  saveLocalTasks()
  newTaskName.value = ''
  fetchTasks()
}

const updateTask = async (task) => {
  const title = task.title.trim()
  if (!title) {
    task.title = 'Untitled Task'
    return
  }

  try {
    const updatedTask = { ...task, title }
    await api.put(`/todos/${task.id}`, updatedTask)
    const index = tasks.value.findIndex(t => t.id === task.id)
    if (index !== -1) {
      tasks.value.splice(index, 1, updatedTask)
    }
  } catch (err) {
    console.error('Failed to update task:', err)
    $q.notify({
      type: 'negative',
      message: 'Failed to update task'
    })
  }
}

const saveLocalTasks = () => {
  localStorage.setItem('localTasks', JSON.stringify(localTasks.value))
}

const loadFinishedTasks = () => {
  const savedFinished = localStorage.getItem('finishedTasks')
  if (savedFinished) {
    try {
      finishedTasks.value = JSON.parse(savedFinished)
    } catch (e) {
      console.error('Failed to parse finished tasks:', e)
      finishedTasks.value = []
    }
  }
}

const saveFinishedTasks = () => {
  localStorage.setItem('finishedTasks', JSON.stringify(finishedTasks.value))
}

const confirmDelete = (id, fromFinished = false) => {
  deleteTaskId.value = id
  deleteFromFinished.value = fromFinished
  confirmDeleteDialog.value = true
}

const deleteTask = async (id, fromFinished = false) => {
  try {
    if (fromFinished) {
      finishedTasks.value = finishedTasks.value.filter(t => t.id !== id)
      saveFinishedTasks()
    } else {
      const localIndex = localTasks.value.findIndex(t => t.id === id)
      if (localIndex !== -1) {
        localTasks.value.splice(localIndex, 1)
        saveLocalTasks()
      } else {
        await api.delete(`/todos/${id}`)
      }
    }

    fetchTasks()
  } catch (err) {
    console.error('Failed to delete task:', err)
    $q.notify({
      type: 'negative',
      message: 'Failed to delete task'
    })
  }
}

const markTaskAsFinished = async (task) => {
  const finished = { ...task, completed: true }

  try {
    const localIndex = localTasks.value.findIndex(t => t.id === task.id)

    if (localIndex !== -1) {
      localTasks.value.splice(localIndex, 1)
      saveLocalTasks()
    } else {
      await api.put(`/todos/${task.id}`, finished)
    }

    finishedTasks.value.unshift(finished)
    saveFinishedTasks()
    fetchTasks()

  } catch (err) {
    console.error('Failed to mark task as finished:', err)
    $q.notify({
      type: 'negative',
      message: 'Failed to mark task as finished'
    })
  }
}
</script>

