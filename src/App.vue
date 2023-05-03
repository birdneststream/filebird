<template>
  <div class="flex justify-center items-center h-screen">
    <div class="bg-gray-900 p-6 rounded-md max-w-screen-lg w-2/6">
      <h2 class="text-lg font-medium text-gray-300 mb-4">Filehole.org</h2>

      <div class="flex flex-col space-y-4">
        <label
          for="file-upload"
          :disabled="uploading"
          class="cursor-pointer flex items-center justify-center rounded-md bg-gray-800 hover:bg-gray-700 transition-colors duration-300 py-3 px-4 text-sm font-medium text-gray-300 max-w-screen-lg w-full"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-6 w-6 mr-2"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M12 6v6m0 0v6m0-6h6m-6 0H6"
            />
          </svg>
          Select Files
        </label>
        <div class="flex flex-col">
          <button
            :disabled="uploading || !this.state.files.length"
            class="max-w-screen-lg w-full bg-blue-500 hover:bg-blue-400 transition-colors duration-300 py-2 px-4 rounded-t text-sm font-medium text-gray-300 disabled:bg-blue-950"
            @click="uploadFiles()"
          >
            {{ uploading ? 'Uploading...' : 'Upload' }}
          </button>
          <button
            :disabled="uploading"
            :class="`max-w-screen-lg w-full bg-blue-500 hover:bg-blue-400 transition-colors duration-300 py-2 px-4 text-sm font-medium text-gray-300 disabled:bg-blue-950`"
            @click="resetState()"
          >
            Clear
          </button>
          <button
            class="max-w-screen-lg w-full bg-blue-500 hover:bg-blue-400 transition-colors duration-300 py-2 px-4 rounded-b text-sm font-medium text-gray-300 disabled:bg-blue-950"
            @click="state.showOptions = !state.showOptions"
          >
            Options
          </button>
          <!-- two input boxes for url length and expiry,  and url length must be between 5 and 169 -->
          <div class="flex flex-wrap p-2" v-if="state.showOptions">
            <div class="flex flex-col space-y-2 flex-grow">
              <label for="url-length" class="text-gray-300 text-center">URL Length</label>
              <input
                id="url-length"
                type="number"
                min="5"
                max="169"
                v-model="state.urllen"
                class="bg-gray-800 rounded-md py-2 px-4 text-gray-300"
              />
              <!-- dropdown select for expiry can only have the values "3600","18000","86400","140400","172800","248400","432000" -->
              <label for="expiry" class="text-gray-300 text-center">Expiry</label>
              <select
                id="expiry"
                v-model="state.expiry"
                class="bg-gray-800 rounded-md py-2 px-4 text-gray-300"
              >
                <option value="3600">1 hour</option>
                <option value="18000">5 hours</option>
                <option value="86400" selected="">1 day</option>
                <option value="140400">39 hours</option>
                <option value="172800">2 days</option>
                <option value="248400">69 hours</option>
                <option value="432000">5 days</option>
              </select>
            </div>
          </div>
        </div>

        <input id="file-upload" type="file" class="hidden" multiple @change="handleFileUpload" />
        <div v-if="state.files.length" class="space-y-2 overflow-y-scroll">
          <div
            v-for="(file, index) in state.files"
            :key="index"
            class="flex items-center justify-between bg-gray-800 rounded-md py-2 px-4"
          >
            <div class="flex items-center space-x-2 w-6">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-6 w-6 text-gray-400"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M6 18L18 6M6 6l12 12"
                />
              </svg>
              <span class="text-gray-300"
                >{{ file.name }} {{ (file.size / 1024).toFixed(2) }}kb</span
              >
            </div>
            <button
              class="text-red-500 hover:text-red-400 transition-colors duration-300 w-8 h-8 rounded-full flex items-center justify-center"
              v-if="!uploading"
              @click="removeFile(index)"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M5.293 5.293a1 1 0 011.414 0L10 8.586l3.293-3.293a1 1 0 111.414 1.414L11.414 10l3.293 3.293a1 1 0 01-1.414 1.414L10 11.414l-3.293 3.293a1 1 0 01-1.414-1.414L8.586 10 5.293 6.707a1 1 0 010-1.414z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          </div>
        </div>

        <!-- area to show uploaded files -->
        <div v-if="state.uploaded.length" class="space-y-2 border-t-2 overflow-y-scroll">
          <div
            v-for="(file, index) in state.uploaded"
            :key="index"
            class="flex items-center justify-between bg-gray-800 rounded-md py-2 px-4 max-w-screen-lg w-full"
          >
            <a
              :href="file.response"
              target="_blank"
              class="text-blue-500 hover:text-blue-400 transition-colors duration-300 rounded-full flex items-center justify-center overflow-hidden"
            >
              <div class="flex items-center">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-6 w-6 text-green-400"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path
                    fill-rule="evenodd"
                    d="M3.293 10.707a1 1 0 010-1.414l5-5a1 1 0 011.414 0l5 5a1 1 0 11-1.414 1.414L10 6.414l-4.293 4.293a1 1 0 01-1.414 0z"
                    clip-rule="evenodd"
                  />
                  <path
                    fill-rule="evenodd"
                    d="M10 18a1 1 0 100-2 1 1 0 000 2z"
                    clip-rule="evenodd"
                  />
                </svg>
                <span class="text-gray-300">{{ file.file }} - {{ file.response }}</span>
              </div>
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      state: {
        files: [],
        uploaded: [],

        expiry: '432000',
        urllen: '5',
        showOptions: false
      },
      uploading: false
    }
  },
  created() {
    if (localStorage.getItem('filehole')) {
      this.state = JSON.parse(localStorage.getItem('filehole'))
    }
  },

  watch: {
    state: {
      handler() {
        localStorage.setItem('filehole', JSON.stringify(this.state))
      },
      deep: true
    }
  },
  methods: {
    resetState() {
      this.state = {
        files: [],
        uploaded: [],
        expiry: '432000',
        urllen: '5'
      }
    },
    handleFileUpload(event) {
      const files = event.target.files
      for (let i = 0; i < files.length; i++) {
        const file = files[i]
        this.state.files.push(file)
      }
    },
    removeFile(index) {
      this.state.files.splice(index, 1)
    },
    async uploadFiles() {
      this.uploading = true
      const formData = new FormData()
      const expiry = this.state.expiry
      const url_len = this.state.urllen
      let files = [...this.state.files]
      for (let i = 0; i < files.length; i++) {
        const file = files[i]
        formData.append('file', file)
        formData.append('expiry', expiry)
        formData.append('url_len', url_len)

        try {
          const response = await fetch('https://filehole.org/', {
            method: 'POST',
            body: formData
          })

          const data = await response.text()
          this.state.uploaded.push({ file: file.name, response: data.trim() })
          this.state.files.shift()
        } catch (error) {
          console.error(`Error uploading file ${i + 1}: ${error}`)
        }
      }
      this.uploading = false
    }
  }
}
</script>
