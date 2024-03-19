<script setup lang="ts">
import { onMounted, ref, type Ref } from 'vue'

interface Task {
  id: number
  title: string
  isComplete: boolean
}

const inputRef = ref<HTMLInputElement | null>(null) // Ref for input element
const allTasks: Ref<Task[]> = ref([])
const newTask = ref<Task>({
  id: Math.floor(Math.random() * 10000),
  title: '',
  isComplete: false
})
const isEdit = ref(false)
let editedTaskId: number | null = null // Store the id of the task being edited

onMounted(() => {
  console.log('inp', inputRef.value)
  inputRef.value?.focus() // Ensure inputRef has a value before accessing its methods
})

const addTask = () => {
  if (newTask.value.title.trim() !== '') {
    if (!isEdit.value) {
      allTasks.value.push({ ...newTask.value })
      newTask.value.id = Math.floor(Math.random() * 10000) // Generate new id for next task
      newTask.value.isComplete = false
      newTask.value.title = ''
    } else {
      const editedTaskIndex = allTasks.value.findIndex((task) => task.id === editedTaskId)
      if (editedTaskIndex !== -1) {
        allTasks.value[editedTaskIndex].title = newTask.value.title
      }
      isEdit.value = false
      editedTaskId = null
      newTask.value.title = ''
    }
  }
}

const deleteTask = (task: Task) => {
  allTasks.value = allTasks.value.filter((t: Task) => t.id !== task.id)
}

const handleDone = (task: Task) => {
  allTasks.value = allTasks.value.map((ele: Task) => {
    if (task.id === ele.id) {
      return { ...ele, isComplete: true }
    }
    return ele
  })
}

const handleEdit = (task: Task) => {
  isEdit.value = true
  editedTaskId = task.id // Store the id of the task being edited
  newTask.value.title = task.title
  inputRef.value?.focus() // Focus on input field when editing
}
</script>

<template>
  <div class="flex w-full h-screen items-center justify-center">
    <div class="p-4 bg-slate-800 rounded-md w-1/2">
      <div class="top flex items-center gap-2 w-full">
        <input
          class="p-2 rounded-md outline-none w-full"
          type="text"
          placeholder="Task "
          @keyup.enter="addTask"
          v-model="newTask.title"
          ref="inputRef"
        />
        <button
          class="py-2 px-4 bg-white font-semibold text-slate-900 font-sans rounded-md hover:bg-slate-100"
          @click.prevent="addTask"
        >
          {{ isEdit ? 'Save' : 'Add' }}
        </button>
      </div>
      <!-- all tasks -->
      <div
        v-show="allTasks.length"
        class="bg-slate-50 mt-5 p-2 rounded-sm max-h-96 overflow-y-auto"
      >
        <ul>
          <li
            class="flex items-center justify-between gap-1 border-b pb-1"
            v-for="(task, index) in allTasks"
            :key="index"
            @click="handleDone(task)"
          >
            <span :class="task.isComplete === true ? 'text-green-500 line-through' : 'text-black'">
              {{ task.title }}
            </span>
            <div class="flex items-center">
              <span
                @click.stop="handleEdit(task)"
                class="text-green-500 font-semibold hover:bg-green-200 px-2 rounded-sm text-sm cursor-pointer"
                >{{ isEdit && editedTaskId === task.id ? 'Cancel' : 'Edit' }}</span
              >
              <span
                @click.stop="deleteTask(task)"
                class="text-red-500 font-semibold hover:bg-red-200 px-2 rounded-sm text-sm cursor-pointer"
                >Delete</span
              >
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
