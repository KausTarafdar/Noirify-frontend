{
  <template>
    <div class="min-h-screen bg-gray-100 p-8">
      <div class="max-w-4xl mx-auto">
        <h1 class="text-3xl font-bold text-gray-800 mb-8">Image & Video Processor</h1>
        
        <!-- Upload Section -->
        <div 
          class="border-2 border-dashed rounded-lg p-8 mb-8 text-center transition-colors"
          :class="{ 'border-blue-500 bg-blue-50': isDragging, 'border-gray-300 hover:border-gray-400': !isDragging }"
          @dragover.prevent="handleDragOver"
          @dragleave.prevent="handleDragLeave"
          @drop.prevent="handleDrop"
        >
          <input
            type="file"
            ref="fileInput"
            @change="handleFileChange"
            accept="image/*,video/*"
            class="hidden"
          />
          <div class="flex flex-col items-center gap-4">
            <div class="p-4 bg-blue-100 rounded-full">
              <Upload class="w-8 h-8 text-blue-600" />
            </div>
            <div>
              <p class="text-lg font-medium">Drag and drop your file here</p>
              <p class="text-gray-500">or</p>
              <button
                @click="$refs.fileInput.click()"
                class="mt-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
              >
                Browse Files
              </button>
            </div>
            <p class="text-sm text-gray-500">Supports images and videos</p>
          </div>
        </div>

        <!-- Preview Section -->
        <div v-if="preview && processed" class="bg-white rounded-lg shadow-lg p-6">
          <div 
            ref="container"
            class="relative h-[400px] mb-6 overflow-hidden rounded-lg"
            @mousemove="handleSliderMove"
          >
            <!-- Processed Image (Left) -->
            <div
              class="absolute top-0 left-0 h-full w-full"
              :style="{
                backgroundImage: `url(${processed})`,
                backgroundSize: 'cover',
                backgroundPosition: 'center',
                filter: `brightness(${controls.brightness}%) contrast(${controls.contrast}%) grayscale(100%)`
              }"
            />
            
            <!-- Original Image (Right) -->
            <div
              class="absolute top-0 left-0 h-full"
              :style="{
                width: `${sliderPosition}%`,
                backgroundImage: `url(${preview})`,
                backgroundSize: 'cover',
                backgroundPosition: 'center'
              }"
            />
            
            <!-- Slider -->
            <div
              class="absolute top-0 bottom-0 w-1 bg-white cursor-ew-resize"
              :style="{ left: `${sliderPosition}%` }"
            >
              <div class="absolute top-1/2 -translate-y-1/2 -translate-x-1/2 w-8 h-8 bg-white rounded-full shadow-lg flex items-center justify-center">
                <SlidersHorizontal class="w-4 h-4 text-gray-600" />
              </div>
            </div>
          </div>

          <!-- Controls -->
          <div class="space-y-4">
            <div v-for="control in ['brightness', 'contrast', 'sharpness']" :key="control">
              <label class="block text-sm font-medium text-gray-700 mb-1">
                {{ control.charAt(0).toUpperCase() + control.slice(1) }}
              </label>
              <input
                type="range"
                min="0"
                max="200"
                :value="controls[control]"
                @input="handleControlChange(control, $event.target.value)"
                class="w-full"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>

  <script setup lang="ts">
  import { ref, reactive } from 'vue'
  import { Upload, SlidersHorizontal } from 'lucide-vue-next'

  interface ImageControls {
    brightness: number
    contrast: number
    sharpness: number
  }

  const fileInput = ref<HTMLInputElement | null>(null)
  const container = ref<HTMLDivElement | null>(null)
  const preview = ref<string | null>(null)
  const processed = ref<string | null>(null)
  const sliderPosition = ref(50)
  const isDragging = ref(false)
  const controls = reactive<ImageControls>({
    brightness: 100,
    contrast: 100,
    sharpness: 100
  })

  const handleFileChange = (e: Event) => {
    const target = e.target as HTMLInputElement
    const file = target.files?.[0]
    if (file) {
      processFile(file)
    }
  }

  const processFile = async (file: File) => {
    // Create preview
    const url = URL.createObjectURL(file)
    preview.value = url

    // Send to backend for processing
    const formData = new FormData()
    formData.append('file', file)

    try {
      // Replace with your actual backend URL
      const response = await fetch('http://your-backend-url/process', {
        method: 'POST',
        body: formData
      })

      if (response.ok) {
        const blob = await response.blob()
        processed.value = URL.createObjectURL(blob)
      } else {
        console.error('Failed to process file')
        // For now, use the same image as preview for demonstration
        processed.value = url
      }
    } catch (error) {
      console.error('Error processing file:', error)
      // For now, use the same image as preview for demonstration
      processed.value = url
    }
  }

  const handleDragOver = (e: DragEvent) => {
    isDragging.value = true
  }

  const handleDragLeave = () => {
    isDragging.value = false
  }

  const handleDrop = (e: DragEvent) => {
    isDragging.value = false
    const file = e.dataTransfer?.files[0]
    if (file) {
      processFile(file)
    }
  }

  const handleSliderMove = (e: MouseEvent) => {
    if (container.value) {
      const rect = container.value.getBoundingClientRect()
      const x = e.clientX - rect.left
      const position = (x / rect.width) * 100
      sliderPosition.value = Math.min(Math.max(position, 0), 100)
    }
  }

  const handleControlChange = (control: keyof ImageControls, value: string) => {
    controls[control] = Number(value)
  }
  </script>}