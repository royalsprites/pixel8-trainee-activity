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
        />
        <q-btn
          color="primary"
          @click="addTask"
        >
          <div class="row items-center no-wrap">
            <span class="q-mr-sm">ADD TASK</span>
            <q-icon name="add" />
          </div>
        </q-btn>
      </div>

      <q-separator />

      <q-card-section>
        <div class="row justify-between items-center q-mb-sm">
          <div class="text-subtitle1 text-weight-medium">Your Tasks</div>
          <q-btn
            v-if="selectedTasks.length > 0"
            color="positive"
            @click="moveToFinished"
          >
            <div class="row items-center no-wrap">
              <span class="q-mr-sm self-center">MARK AS FINISHED</span>
              <q-icon name="check" />
            </div>
          </q-btn>
        </div>
        <q-list bordered separator>
          <q-item
            v-for="task in tasks"
            :key="task.id"
            clickable
            class="q-hoverable"
          >
            <q-item-section side>
              <q-checkbox v-model="selectedTasks" :val="task.id" />
            </q-item-section>

            <q-item-section>
              <q-input
                v-if="editingTaskId === task.id"
                v-model="editingTaskTitle"
                autofocus
                dense
                borderless
                @keyup.enter="saveEdit(task)"
                @blur="saveEdit(task)"
              />
              <div v-else>
                {{ task.title }}
              </div>
            </q-item-section>

            <q-item-section side>
              <div class="row q-gutter-xs">
                <q-btn
                  :icon="editingTaskId === task.id ? 'check' : 'edit'"
                  :color="editingTaskId === task.id ? 'positive' : 'warning'"
                  flat
                  @click="onEditButtonClick(task)"
                />
                <q-btn
                  flat
                  icon="delete"
                  color="negative"
                  @click="deleteTask(task.id)"
                />
              </div>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { api } from 'boot/axios'

const tasks = ref([])
const newTaskName = ref('')
const editingTaskId = ref(null)
const editingTaskTitle = ref('')
const selectedTasks = ref([])
const finishedTasks = ref([])

onMounted(() => {
  fetchTasks()
  const savedFinishedTasks = localStorage.getItem('finishedTasks')
  if (savedFinishedTasks) {
    finishedTasks.value = JSON.parse(savedFinishedTasks)
  }
})

const fetchTasks = async () => {
  try {
    const res = await api.get('/todos')
    tasks.value = res.data.filter(task => !task.completed)
  } catch (error) {
    console.error('Error fetching tasks:', error)
    tasks.value = []
  }
}

const addTask = async () => {
  if (!newTaskName.value.trim()) return

  try {
    const res = await api.post('/todos', {
      title: newTaskName.value,
      completed: false
    })
    tasks.value.unshift({
      id: res.data.id,
      title: res.data.title,
      completed: res.data.completed
    })
    newTaskName.value = ''
    editingTaskId.value = null
    editingTaskTitle.value = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const startEdit = (task) => {
  editingTaskId.value = task.id
  editingTaskTitle.value = task.title || ''
}

const saveEdit = async (task) => {
  if (!editingTaskTitle.value.trim()) {
    editingTaskId.value = null
    return
  }

  try {
    const updatedTask = {
      ...task,
      title: editingTaskTitle.value
    }

    await api.put(`/todos/${task.id}`, updatedTask)

    const index = tasks.value.findIndex(t => t.id === task.id)
    if (index !== -1) {
      tasks.value.splice(index, 1, updatedTask)
    }

    editingTaskId.value = null
    editingTaskTitle.value = ''
  } catch (error) {
    console.error('Error updating task:', error)
  }
}

const deleteTask = async (id) => {
  try {
    await api.delete(`/todos/${id}`)
    tasks.value = tasks.value.filter(task => task.id !== id)

    if (editingTaskId.value === id) {
      editingTaskId.value = null
      editingTaskTitle.value = ''
    }
  } catch (error) {
    console.error('Error deleting task:', error)
  }
}

const moveToFinished = async () => {
  const tasksToMove = tasks.value.filter(task => selectedTasks.value.includes(task.id))
  const existingIds = new Set(finishedTasks.value.map(t => t.id))
  const newFinished = tasksToMove.filter(task => !existingIds.has(task.id))
  finishedTasks.value.push(...newFinished)
  localStorage.setItem('finishedTasks', JSON.stringify(finishedTasks.value))

  for (const id of selectedTasks.value) {
    try {
      await api.delete(`/todos/${id}`)
    } catch (err) {
      console.error('Error deleting task:', err)
    }
  }
  tasks.value = tasks.value.filter(task => !selectedTasks.value.includes(task.id))
  selectedTasks.value = []
}

const onEditButtonClick = (task) => {
  if (editingTaskId.value === task.id) {
    saveEdit(task)
  } else {
    startEdit(task)
  }
}
</script>
