<template>
  <transition
    enter-active-class="transform transition ease-out duration-300"
    enter-class="-translate-y-full opacity-0"
    enter-to-class="translate-y-0 opacity-100"
    leave-active-class="transform transition ease-in duration-200"
    leave-class="translate-y-0 opacity-100"
    leave-to-class="-translate-y-full opacity-0"
  >
    <div class="fixed top-0 left-0 w-full z-10 pt-6">
      <div class="max-w-7xl mx-auto px-4 sm:px-6">
        <div class="rounded-b-lg shadow-lg">
          <div class="rounded-lg shadow-xs bg-white">
            <div class="flex items-center  justify-between p-4">
              <div class="flex items-center space-x-2">
                <button
                  type="button"
                  class="px-2 py-2 flex-shrink-0 truncate rounded leading-tight tracking-wide text-gray-500 font-semibold hover:bg-gray-200 focus:outline-none focus:bg-gray-200"
                  @click="$emit('unselect-all')"
                >
                  <svg
                    class="h-6 w-6 text-gray-500"
                    xmlns="http://www.w3.org/2000/svg"
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
                </button>
                <span class="font-medium leading-4 text-gray-800"
                  >{{ itemsCount }} items selected</span
                >
              </div>
              <div class="flex items-center space-x-2">
                <button
                  type="button"
                  v-show="itemsCount === 1"
                  @click="$emit('rename-item')"
                  class="px-2 py-2 truncate rounded leading-tight tracking-wide text-gray-500 font-semibold hover:bg-gray-200 focus:outline-none focus:bg-gray-200"
                >
                  <svg
                    class="h-6 w-6 text-gray-500"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"
                    />
                  </svg>
                </button>
                <button
                  type="button"
                  class="px-2 py-2 truncate rounded leading-tight tracking-wide text-gray-500 font-semibold hover:bg-gray-200 focus:outline-none focus:bg-gray-200"
                  @click="deleteItems()"
                >
                  <svg
                    class="h-6 w-6 text-gray-500"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                    />
                  </svg>
                </button>
                <button
                  type="button"
                  class="px-2 py-2 truncate rounded leading-tight tracking-wide text-gray-500 font-semibold hover:bg-gray-200 focus:outline-none focus:bg-gray-200"
                  @click="downloadItems()"
                >
                  <svg
                    class="h-6 w-6 text-gray-500"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
                    />
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import axios from 'axios'
import mime from 'mime-types'
import fileDownload from 'js-file-download'

export default {
  name: 'SelectedItemsActionsBar',
  props: ['items'],
  computed: {
    itemsCount() {
      return this.items.files.length + this.items.folders.length
    },
  },
  methods: {
    async deleteItems() {
      try {
        if (this.items.files.length) await this.deleteAction('files')
        if (this.items.folders.length) await this.deleteAction('folders')

        this.$store.commit('removeDeletedItems', this.items)
        this.$emit('unselect-all')
      } catch (error) {
        console.log('Error while deleting items', error)
      }
    },
    async deleteAction(type) {
      let items = []
      this.items[type].forEach(item => {
        items.push(item.id)
      })

      const response = await axios.delete(`https://api.swap.test/${type}/delete`, {
        data: {
          items: items,
        },
      })

      return response
    },
    async downloadItems() {
      const files = this.items.files.map(file => file.id)
      const folders = this.items.folders.map(folder => folder.id)

      try {
        const { data, headers } = await axios.post(
          `https://api.swap.test/items/download`,
          {
            files: files,
            folders: folders,
          },
          { responseType: 'blob' }
        )

        let filename = `swap-download--${new Date().getTime()}.zip`

        // Replace filename with item name if only one file is being downloaded
        if (files.length === 1 && folders.length === 0) {
          filename = this.items.files[0].name.replace(/\.[^/.]+$/, '')
          filename = `${filename}.${mime.extension(headers['content-type'])}`
        }

        fileDownload(data, filename)
        this.$emit('unselect-all')
      } catch (error) {
        console.log('Error', error)
      }
    },
  },
}
</script>
