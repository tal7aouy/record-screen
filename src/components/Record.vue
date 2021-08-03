<template>
  <div class="h-screen w-full flex-col flex justify-between items-center">
    <div class="flex mx-auto">
      <button
        class="bg-blue-500 text-white px-6 py-2 rounded font-medium mx-3 hover:bg-blue-600 transition duration-200 each-in-out shadow-md"
        v-if="!isRecording"
        @click="startRecord"
      >
        Start Recording 🎥
      </button>
      <button
        class="bg-red-500 text-white px-6 py-2 rounded font-medium mx-3 hover:bg-red-600 transition duration-200 each-in-out shadow-md"
        v-else
        @click="stopRecord"
      >
        Stop Recording ❌
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Record',
  data() {
    return {
      isRecording: false,
      options: { mimType: 'video/webm; codecs=vp9' },
      displayOptions: {
        video: {
          cursor: 'always',
        },
        audio: {
          echoCancellation: true,
          noiseSuppression: true,
          sampleRate: 44100,
        },
      },
      mediarecorder: {},
      stream: {},
      recordChunks: [],
    }
  },
  methods: {
    download() {
      let blob = new Blob(this.recordChunks, {
        type: 'video/webm',
      })
      let url = URL.createObjectURL(blob)
      let a = document.createElement('a')
      document.body.appendChild(a)
      a.style = 'display: none'
      a.href = url
      let name = new Date().toUTCString()
      a.download = `${name}.webm`
      a.click()
      URL.revokeObjectURL(url)
      this.recordChunks = []
      this.showNotification()
    },
    showNotification() {
      const notify = new Notification('successfuly', {
        body: 'you have been recorded the video successfully',
        icon: 'https://img.icons8.com/flat-round/64/000000/record.png',
      })
      setTimeout(notify.close(), 1 * 1000)
    },
    handleDataAvailable(event) {
      if (event.data.size > 0) {
        this.recordChunks.push(event.data)
        this.isRecording = false

        this.download()
      } else {
        //...
      }
    },
    stopRecord() {
      this.mediarecorder.stop()
    },
    async startRecord() {
      try {
        this.stream = await navigator.mediaDevices.getDisplayMedia(
          this.displayOptions
        )
        this.mediarecorder = new MediaRecorder(this.stream, this.options)
        this.mediarecorder.ondataavailable = this.handleDataAvailable
        this.mediarecorder.start()
        this.isRecording = true
      } catch (error) {
        this.isRecording = false
        alert(error)
      }
    },
    process(permission) {
      if (permission === 'granted') {
        alert('granted')
      } else {
        console.log('denied')
      }
    },
  },
  created() {
    Notification.requestPermission((permission) => {
      this.process(permission)
    }).catch((permission) => {
      this.process(permission)
    })
  },
}
</script>
