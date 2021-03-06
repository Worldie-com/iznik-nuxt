<template>
  <div>
    <div v-if="!showVideo">
      <div v-if="!result && !processing && !uploading">
        <h1>
          Booktastic Proof of Concept
        </h1>
        <b-card no-body>
          <b-card-body>
            <p>
              Take a closeup of a bookshelf.  You're aiming for something like this:
            </p>
            <b-img src="~/static/booktastic.jpg" thumbnail class="smallimg mb-2" />
            <p class="text-danger">
              Please read these tips:
            </p>
            <ul>
              <li>
                Take a <strong>close-up</strong> of <strong>just one shelf</strong>.  If you can see two shelves, go closer.
              </li>
              <li>
                <strong>Straight</strong> - not at a rakish angle.
              </li>
              <li>
                Your picture should show <strong>about 10 books</strong>.  If it's more than that, go closer.
              </li>
              <li>
                <strong>Avoid glare</strong> / reflections.
              </li>
              <li>
                Squidge the books up so they're not slanted.
              </li>
              <li>
                Paperbacks work better than hardbacks.
              </li>
              <li>
                Fiction works better than non-fiction.
              </li>
            </ul>
            <p>
              We'll try to identify them.  See some sample results and background info <nuxt-link to="/booktastic/results">
                here
              </nuxt-link>.
            </p>
          </b-card-body>
        </b-card>
        <div class="d-flex justify-content-between flex-wrap">
          <b-btn
            v-if="!showVideo"
            variant="secondary"
            size="lg"
            class="mt-2"
            @click="takePhoto"
          >
            <v-icon name="camera" />&nbsp;Take photo
          </b-btn>
          <b-btn
            variant="primary"
            size="lg"
            class="mt-2"
            @click="startUpload"
          >
            <v-icon name="camera" />&nbsp;Upload photo
          </b-btn>
        </div>
      </div>
      <b-btn v-if="result" variant="secondary" class="mt-2 mb-2" size="lg" @click="again">
        Try Again
      </b-btn>
      <client-only>
        <BooktasticResult
          v-if="result"
          :id="requestId"
          :photo="photo"
          :result="result"
          :width="width"
          :height="height"
        />
      </client-only>
      <b-row v-if="processing">
        <b-col cols="12" lg="4" offset-lg="4">
          <NoticeMessage variant="info">
            <div class="w-100 d-flex justify-content-center text-center">
              <div class="d-flex flex-column">
                <p>
                  We won't get them all - right now around half is pretty good going.
                </p>
                <p>
                  It's quite slow - upto 60 seconds to process.  Don't worry about that - it'll
                  get faster later.
                </p>
                <h4>Crunching...</h4>
                <div>
                  <b-img-lazy src="~/static/loader.gif" alt="Loading" />
                </div>
              </div>
            </div>
          </NoticeMessage>
        </b-col>
      </b-row>
    </div>
    <div v-else>
      <div class="video-container">
        <div class="w-100 d-flex justify-content-around video-content">
          <SpinButton
            v-if="showVideo"
            variant="primary"
            name="camera"
            label="Take Photo"
            :handler="capture"
            size="lg"
            class="mt-2 top"
          />
        </div>
        <video
          v-show="showVideo"
          v-if="!uploading && !processing && !result"
          id="video"
          ref="video"
          class="video-content"
          :width="width"
          :height="height"
          autoplay
        />
        <fabric-canvas
          ref="canvas"
          background-color="transparent"
          class="position-absolute canvas"
          :width="width"
          :height="height"
        >
          <fabric-rectangle
            v-for="(guideline, index) in guidelines"
            :id="'guideline-' + index"
            :key="'guideline-' + index + '-' + bump"
            fill="transparent"
            :top="guideline.top"
            :left="guideline.left"
            :height="guideline.height"
            :width="guideline.width"
            stroke="green"
            :stroke-width="3"
            lock-movement-x
            lock-movement-y
            lock-scaling-x
            lock-scaling-y
          />
        </fabric-canvas>
        <b-img ref="preview" :src="preview" fluid class="video-content" />
      </div>
      <div>
        <b-form-input
          ref="range"
          v-model="rangeVal"
          type="range"
          :min="rangeMin"
          :max="rangeMax"
          :step="rangeStep"
          class="top"
          @change="setFocus"
        />
      </div>
    </div>
    <file-pond
      v-if="uploading"
      ref="pond"
      name="photo"
      :allow-multiple="false"
      accepted-file-types="image/jpeg, image/png, image/gif, image/jpg"
      :files="myFiles"
      image-crop-aspect-ratio="1"
      label-idle="<span class=&quot;btn btn-primary&quot;>&nbsp;Upload&nbsp;Photo </span>"
      :server="{ process, revert, restore, load, fetch }"
      @init="photoInit"
      @processfile="processed"
    />
  </div>
</template>

<script>
import 'filepond/dist/filepond.min.css'
import 'filepond-plugin-image-preview/dist/filepond-plugin-image-preview.min.css'
import vueFilePond from 'vue-filepond'
import FilePondPluginFileValidateType from 'filepond-plugin-file-validate-type'
import FilePondPluginImagePreview from 'filepond-plugin-image-preview'
import FilePondPluginImageTransform from 'filepond-plugin-image-transform'
import FilePondPluginImageResize from 'filepond-plugin-image-resize'
import FilePondPluginImageExifOrientation from 'filepond-plugin-image-exif-orientation'
import SpinButton from '@/components/SpinButton'
import BooktasticResult from '@/components/BooktasticResult'
import NoticeMessage from '@/components/NoticeMessage'
import waitForRef from '@/mixins/waitForRef'

const a = require('axios')
const axios = a.create({
  timeout: 300000
})

const FilePond = vueFilePond(
  FilePondPluginFileValidateType,
  FilePondPluginImagePreview,
  FilePondPluginImageTransform,
  FilePondPluginImageResize,
  FilePondPluginImageExifOrientation
)

export default {
  head: {
    title: 'Capture',
    script: [{}],
    meta: [
      {
        hid: 'description',
        name: 'description',
        content: 'Capture books from camera'
      }
    ]
  },
  components: { NoticeMessage, BooktasticResult, SpinButton, FilePond },
  mixins: [waitForRef],
  data() {
    return {
      mediaStream: null,
      captureDevice: {},
      videoDevice: null,
      photoCapabilities: null,
      mediaSettings: null,
      rangeVal: null,
      rangeStep: null,
      rangeMin: null,
      rangeMax: null,
      video: {},
      myFiles: [],
      photo: null,
      result: null,
      uploading: false,
      processing: false,
      timeout: 3000,
      requestId: null,
      showVideo: false,
      preview: null,
      previewTimer: null,
      bump: 1,
      frame: null,
      boxes: []
    }
  },
  computed: {
    guidelines() {
      const ret = []

      if (this.frame) {
        const hscale = this.height / this.frame.height
        const wscale = this.width / this.frame.width
        console.log(
          this.frame.height,
          this.frame.width,
          this.height,
          this.width,
          hscale,
          wscale
        )

        this.boxes.forEach(b => {
          console.log('Box', b)
          ret.push({
            top: Math.round(b.boundingBox.top * hscale),
            left: Math.round(b.boundingBox.left * wscale),
            height: Math.round(b.boundingBox.height * hscale),
            width: Math.round(b.boundingBox.width * wscale)
          })
        })
      }

      console.log('Guidelines', ret)
      return ret
    },
    width() {
      return this.bump ? window.innerWidth : 0
    },
    height() {
      return this.bump ? window.innerHeight - 100 : 0
    }
  },
  mounted() {
    window.addEventListener('resize', () => {
      // This keeps our guidelines right.
      console.log('Resized')
      this.bump++
    })
  },
  beforeDestroy() {
    this.stopCamera()

    if (this.previewTimer) {
      clearTimeout(this.previewTimer)
    }
  },
  methods: {
    readFileAsync(blob) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()

        reader.onloadend = () => {
          resolve(reader.result)
        }

        reader.onerror = reject

        reader.readAsDataURL(blob)
      })
    },
    async upload(file) {
      const base64data = await this.readFileAsync(file)
      const formData = new FormData()
      formData.append('photo', base64data)
      let response = null
      this.photo = base64data

      response = await axios.post(process.env.API + '/catalogue', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })

      this.processing = true
      const rsp = response.data
      console.log('Response', rsp)

      if (rsp.ret === 0) {
        this.requestId = rsp.id
        console.log('Queued as', this.requestId)
        this.checkResult()
      }
    },
    async checkResult() {
      console.log('Check request', this.requestId)
      const response = await axios.get(process.env.API + '/catalogue', {
        params: {
          id: this.requestId
        }
      })

      const rsp = response.data

      if (rsp.ret === 0 && rsp.spines && rsp.spines.length) {
        console.log('Completed')
        this.processing = false
        this.result = rsp
      } else {
        this.timeout--

        if (--this.timeout > 0) {
          setTimeout(this.checkResult, 1000)
        } else {
          console.log('Gave up')
        }
      }
    },
    async takePhoto() {
      this.showVideo = true

      this.mediaStream = await navigator.mediaDevices.getUserMedia({
        video: {
          facingMode: 'environment'
        },
        audio: false
      })

      this.previewTimer = setTimeout(this.capture, 1)

      // Set it playing onscreen.
      // this.waitForRef('video', () => {
      this.video = this.$refs.video
      this.video.srcObject = this.mediaStream
      this.video.play()
      // })
    },
    async setFocus() {
      console.log('Set focus', this.rangeVal)
      try {
        this.photoCapabilities = await this.captureDevice.getPhotoCapabilities()
        const trackCapabilities = this.videoDevice.getCapabilities()

        if (trackCapabilities && trackCapabilities.focusDistance) {
          this.videoDevice.applyConstraints({
            advanced: [
              {
                focusMode: 'manual',
                focusDistance: this.rangeVal
              }
            ]
          })
          console.log('Set ok')
          this.mediaSettings = this.videoDevice.getSettings()
          console.log('Media settings', this.mediaSettings)
        }
      } catch (e) {
        console.log('Force focus failed', e)
      }
    },
    async showCapture(blob) {
      const base64data = await this.readFileAsync(blob)
      this.preview = base64data
    },
    async capture() {
      if (this.mediaStream) {
        try {
          const videoTracks = this.mediaStream.getVideoTracks()
          console.log('Video tracks', videoTracks)
          this.videoDevice = videoTracks[0]
          console.log('Video device', this.videoDevice)
          this.captureDevice = new ImageCapture(this.videoDevice)
          console.log('Capture device', this.captureDevice)

          this.frame = await this.captureDevice.grabFrame()
          console.log('Frame', this.frame)
          // eslint-disable-next-line no-undef
          const textDetector = new TextDetector()
          this.boxes = await textDetector.detect(this.frame)
          console.log('Boxes', this.boxes)
        } catch (e) {
          console.log('Capture error', e)
        }

        this.previewTimer = setTimeout(this.capture, 10000)
      } else {
        alert('No device')
      }
    },
    stopCamera() {
      if (this.captureDevice && this.captureDevice.stop) {
        this.captureDevice.stop()
      }
    },
    photoInit: function() {
      if (!this.$refs.pond._pond) {
        // This is the only way of finding out if the browser is supported - see
        // https://github.com/pqina/vue-filepond/issues/136
        this.supported = false
      } else if (this.browse) {
        // We have rendered the filepond instance.  Trigger browse so that they can upload a photo without an
        // extra click.
        this.$refs.pond.browse()
      }
    },
    process(fieldName, file, metadata, load, error, progress, abort) {
      this.upload(file).then(() => {
        this.uploading = false
      })

      return {
        abort: () => {
          // We don't need to do anything - the server will tidy up hanging images.
          abort()
        }
      }
    },
    load(uniqueFileId, load, error) {},
    fetch(url, load, error, progress, abort, headers) {},
    restore(uniqueFileId, load, error, progress, abort, headers) {},
    revert(uniqueFileId, load, error) {},
    processed() {},

    addFile(f) {
      this.$refs.pond.addFile(f)
    },

    detector(source, type) {
      // This function is never executed...
      return new Promise((resolve, reject) => {
        console.log(source, type)
        if (source.name.indexOf('.heic') !== -1) {
          // This is not detected automatically.
          type = 'image/heic'
        }

        resolve(type)
      })
    },
    again() {
      this.photo = null
      this.result = null
      this.timeout = 300
      this.uploading = false
      this.processing = false
    },
    startUpload() {
      this.uploading = true
      this.waitForRef('pond', () => {
        this.$refs.pond.browse()
      })
    }
  }
}
</script>
<style>
body {
  background-color: #f0f0f0;
}

#video {
  background-color: #000000;
}

.smallimg {
  max-width: 200px;
}

.video-container {
  width: 100vw;
  height: calc(100vh - 100px);
  position: relative;
}

.video-content {
  width: 100%;
  /*height: 100%;*/
  position: absolute;
  top: 0;
  left: 0;
}

.top {
  z-index: 2000;
}
</style>
