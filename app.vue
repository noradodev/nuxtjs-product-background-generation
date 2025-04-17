<script setup>
import { ref } from 'vue'
import axios from 'axios'

const prompt = ref('')
const negativePrompt = ref('')
const imageUrl = ref(null)
const inputImageUrl = ref(null)
const selectedFile = ref(null)
const loading = ref(false)

const showToast = ref(false)
const toastMessage = ref('')
const toastType = ref('success')

const showNotification = (message, type = 'success') => {
  toastMessage.value = message
  toastType.value = type
  showToast.value = true
  setTimeout(() => (showToast.value = false), 3000)
}

const handleFileUpload = (event) => {
  selectedFile.value = event.target.files[0]
  inputImageUrl.value = URL.createObjectURL(selectedFile.value)
}

const clearImages = () => {
  inputImageUrl.value = null
  imageUrl.value = null
  selectedFile.value = null
}
console.log(import.meta.env.VITE_BACKEND_API)


const submitForm = async () => {
  if (!selectedFile.value || !prompt.value) {
    showNotification('Please upload an image and enter a prompt.', 'error')
    return
  }

  const formData = new FormData()
  formData.append('file', selectedFile.value)
  formData.append('prompt', prompt.value)
  formData.append('negative_prompt', negativePrompt.value || 'blurry, CGI, distorted')

  loading.value = true
  imageUrl.value = null
  

  try {
    const response = await axios.post(import.meta.env.VITE_BACKEND_API, formData, {
      responseType: 'blob',
    })

    const blob = new Blob([response.data], { type: 'image/png' })
    imageUrl.value = URL.createObjectURL(blob)
    showNotification('Image generated successfully!')
  } catch (error) {
    showNotification('Failed to generate image.', 'error')
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>


<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-100 via-white to-sky-200 py-20 px-6 flex items-center justify-center">
    <div class="backdrop-blur-lg bg-white/70 border border-white/30 rounded-3xl shadow-2xl p-10 w-full max-w-3xl transition-all duration-300">
      <h1 class="text-4xl font-bold text-center text-gray-800 tracking-tight mb-8">
        AI Product Image Generator
      </h1>

      <form @submit.prevent="submitForm" class="space-y-6">
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-6">
          <div>
            <label class="block text-sm font-medium text-gray-600">Product Image</label>
            <input
              type="file"
              accept="image/*"
              @change="handleFileUpload"
              class="mt-1 block w-full rounded-xl border border-gray-300 shadow-sm p-2 bg-white/80 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-600">Prompt</label>
            <input
              v-model="prompt"
              type="text"
              placeholder="e.g., A clean blue background, studio lighting..."
              class="mt-1 block w-full rounded-xl border border-gray-300 shadow-sm p-2 bg-white/80 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-600">Negative Prompt</label>
            <input
              v-model="negativePrompt"
              type="text"
              placeholder="e.g., blurry, low quality, CGI"
              class="mt-1 block w-full rounded-xl border border-gray-300 shadow-sm p-2 bg-white/80 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>

        <button
          type="submit"
          class="w-full mt-4 bg-gradient-to-r from-blue-500 to-sky-500 hover:from-blue-600 hover:to-sky-600 text-white font-semibold py-3 rounded-xl transition-all shadow-md hover:shadow-lg"
          :disabled="loading"
        >
          <span v-if="loading">⏳ Generating...</span>
          <span v-else>🚀 Generate Image</span>
        </button>

        <button
          type="button"
          v-if="inputImageUrl || imageUrl"
          @click="clearImages"
          class="w-full mt-2 bg-gray-100 hover:bg-gray-200 text-gray-800 font-medium py-2 rounded-xl transition-all"
        >
          🗑️ Clear Images
        </button>
      </form>

      <div v-if="inputImageUrl || imageUrl" class="mt-10 grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <h2 class="text-center text-gray-700 font-medium mb-2">Input Image</h2>
          <img :src="inputImageUrl || 'https://placehold.co/400'" alt="Input" class="rounded-xl shadow-lg w-full" />
        </div>

        <div>
          <h2 class="text-center text-gray-700 font-medium mb-2">Generated Image</h2>
          <div class="relative">
            <div v-if="loading" class="absolute inset-0 flex items-center justify-center bg-white/80 z-10 rounded-xl">
              <svg class="animate-spin h-8 w-8 text-blue-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"></path>
              </svg>
            </div>
            <img
              v-if="imageUrl"
              :src="imageUrl"
              alt="Generated"
              class="rounded-xl shadow-lg w-full"
            />
            <img
              v-else
              src="https://placehold.co/400"
              class="rounded-xl shadow-lg w-full"
              alt="Placeholder"
            />
          </div>
        </div>
      </div>

      <!-- Toast -->
      <div
        v-if="showToast"
        :class="[
          'fixed bottom-6 right-6 px-4 py-3 rounded-lg text-white shadow-lg transition-all',
          toastType === 'success' ? 'bg-green-500' : 'bg-red-500'
        ]"
      >
        {{ toastMessage }}
      </div>
    </div>
  </div>
</template>
