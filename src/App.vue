<template>
  <v-app :theme="isDark ? 'dark' : 'light'">
    <v-container fluid class="fill-height pa-0 ma-0">
      <v-row class="fill-height ma-0" no-gutters>
        <v-col cols="12">
          <v-card class="h-100 w-100 pa-6" elevation="0">
            <v-card-title class="text-h5 d-flex align-center justify-space-between mb-4">
              <div class="d-flex align-center">
                <v-icon class="mr-2">mdi-timer</v-icon>
                Task Timer Tracker
              </div>
              <v-btn icon @click="toggleTheme">
                <v-icon>{{ isDark ? 'mdi-white-balance-sunny' : 'mdi-moon-waning-crescent' }}</v-icon>
              </v-btn>
            </v-card-title>

            <v-text-field
              v-model="newTask"
              label="Новая задача"
              @keyup.enter="addTask"
              append-inner-icon="mdi-plus"
              @click:append-inner="addTask"
              outlined
              dense
              class="mb-6"
            />

            <v-list v-if="tasks.length">
              <v-list-item
                v-for="task in tasks"
                :key="task.id"
                class="mb-3 rounded"
                :class="task.running ? 'bg-blue-lighten-5' : ''"
              >
                <v-list-item-content>
                  <v-list-item-title class="text-body-1 font-weight-medium">
                    {{ task.name }}
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-caption">{{ formatTime(task.time) }}</v-list-item-subtitle>
                </v-list-item-content>

                <v-list-item-action class="d-flex align-center">
                  <v-btn
                    icon
                    :color="task.running ? 'grey' : 'green'"
                    @click="toggleTimer(task)"
                    class="mr-1"
                  >
                    <v-icon>{{ task.running ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                  </v-btn>
                  <v-btn icon color="red" @click="removeTask(task.id)">
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </v-list-item-action>
              </v-list-item>
            </v-list>

            <v-alert v-else type="info" border="start" class="mt-4">
              Пока задач нет — добавь первую!
            </v-alert>

            <v-divider class="my-4" />

            <div class="d-flex justify-space-between align-center">
              <div class="text-subtitle-2">Общее время: {{ formatTime(totalTime) }}</div>
              <v-btn color="red" variant="text" @click="clearAllTasks">Очистить всё</v-btn>
            </div>

          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const newTask = ref('')
const tasks = ref([])
const isDark = ref(false)

const addTask = () => {
  if (!newTask.value.trim()) return
  tasks.value.push({
    id: Date.now(),
    name: newTask.value,
    time: 0,
    running: false,
  })
  newTask.value = ''
  saveTasks()
}

const toggleTimer = (task) => {
  tasks.value.forEach(t => {
    if (t.id !== task.id) t.running = false
  })
  task.running = !task.running
  saveTasks()
}

const removeTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id)
  saveTasks()
}

const clearAllTasks = () => {
  tasks.value = []
  saveTasks()
}

const formatTime = (seconds) => {
  const m = Math.floor(seconds / 60).toString().padStart(2, '0')
  const s = (seconds % 60).toString().padStart(2, '0')
  return `${m}:${s}`
}

const totalTime = computed(() => {
  return tasks.value.reduce((acc, t) => acc + t.time, 0)
})

const toggleTheme = () => {
  isDark.value = !isDark.value
}

const saveTasks = () => {
  localStorage.setItem('task-timer-tasks', JSON.stringify(tasks.value))
}

const loadTasks = () => {
  const data = localStorage.getItem('task-timer-tasks')
  if (data) tasks.value = JSON.parse(data)
}

onMounted(() => {
  loadTasks()
  setInterval(() => {
    const runningTask = tasks.value.find(t => t.running)
    if (runningTask) {
      runningTask.time++
      saveTasks()
    }
  }, 1000)
})
</script>

<style scoped>
.v-card {
  border-radius: 0;
}
</style>