<template>
  <div class="uploader-file" :status="status">
    <slot :file="file" :list="list" :status="status" :paused="paused" :error="error" :response="response"
      :average-speed="averageSpeed" :formated-average-speed="formatedAverageSpeed" :current-speed="currentSpeed"
      :is-complete="isComplete" :is-uploading="isUploading" :size="size" :formated-size="formatedSize"
      :uploaded-size="uploadedSize" :progress="progress" :progress-style="progressStyle"
      :progressing-class="progressingClass" :time-remaining="timeRemaining"
      :formated-time-remaining="formatedTimeRemaining" :type="type" :extension="extension"
      :file-category="fileCategory">
      <div class="task is-flex">
        <div class="task-progress normal" :class="progressingClass" :style="progressStyle"
          v-if="status != 'success'"></div>
        <div class="task-info">
          <div class="cover">
            <div class="folder-cover">
              <img alt="folder" :src="getIconFile(file)" class="folder-icon" />
            </div>
          </div>
          <div class="task-info-wrapper">
            <span class="task-file-name">{{file.name}}</span>
            <div class="task-desc-wrapper">
              <span class="task-desc" v-if="status != 'success'">{{size*progress | renderSize}}/{{formatedSize}}</span>
              <span class="task-desc" v-else>{{formatedSize}}</span>
              <span class="task-dot"></span>
              <span class="task-desc">
                {{ $t(statusText)}}
              </span>
            </div>
          </div>
        </div>
        <div class="action">

          <span data-role="icon" class="action-icon icon" @click="remove" v-if="status != 'success'">
            <b-icon icon="close" size="is-small"></b-icon>
          </span>

          <span data-role="icon" class="action-icon icon" @click="retry" v-if="status == 'error' ">
            <b-icon icon="replay" size="is-small"></b-icon>
          </span>

          <span data-role="icon" class="action-icon icon" @click="resume" v-if="status == 'paused'">
            <b-icon icon="play" size="is-small"></b-icon>
          </span>

          <span data-role="icon" class="action-icon icon" @click="pause" v-if="status == 'uploading'">
            <b-icon icon="pause" size="is-small"></b-icon>
          </span>

        </div>
      </div>
    </slot>
  </div>
</template>

<script>
import { mixin } from '@/mixins/mixin';
import Uploader from 'simple-uploader.js'
import events from '../common/file-events'
import { secondsToStr } from '../common/utils'

const COMPONENT_NAME = 'uploader-file'

export default {
  name: COMPONENT_NAME,
  mixins: [mixin],
  props: {
    file: {
      type: Object,
      default() {
        return {}
      }
    },
    list: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      response: null,
      paused: false,
      error: false,
      averageSpeed: 0,
      currentSpeed: 0,
      isComplete: false,
      isUploading: false,
      size: 0,
      formatedSize: '',
      uploadedSize: 0,
      progress: 0,
      timeRemaining: 0,
      type: '',
      extension: '',
      progressingClass: ''
    }
  },
  computed: {
    fileCategory() {
      const extension = this.extension
      const isFolder = this.file.isFolder
      let type = isFolder ? 'folder' : 'unknown'
      const categoryMap = this.file.uploader.opts.categoryMap
      const typeMap = categoryMap || {
        image: ['gif', 'jpg', 'jpeg', 'png', 'bmp', 'webp'],
        video: ['mp4', 'm3u8', 'rmvb', 'avi', 'swf', '3gp', 'mkv', 'flv'],
        audio: ['mp3', 'wav', 'wma', 'ogg', 'aac', 'flac'],
        document: ['doc', 'txt', 'docx', 'pages', 'epub', 'pdf', 'numbers', 'csv', 'xls', 'xlsx', 'keynote', 'ppt', 'pptx']
      }
      Object.keys(typeMap).forEach((_type) => {
        const extensions = typeMap[_type]
        if (extensions.indexOf(extension) > -1) {
          type = _type
        }
      })
      return type
    },
    progressStyle() {
      const progress = Math.floor(this.progress * 100)
      // const style = `translateX(${Math.floor(progress - 100)}%)`
      return {
        progress: `${progress}%`,
        width: `${progress}%`,
      }
    },
    formatedAverageSpeed() {
      return `${Uploader.utils.formatSize(this.averageSpeed)}/s`
    },
    formateCurrentSpeed() {
      return `${Uploader.utils.formatSize(this.currentSpeed)} / s`
    },
    status() {
      const isUploading = this.isUploading
      const isComplete = this.isComplete
      const isError = this.error
      const paused = this.paused
      if (isComplete) {
        return 'success'
      } else if (isError) {
        return 'error'
      } else if (isUploading) {
        return 'uploading'
      } else if (paused) {
        return 'paused'
      } else {
        return 'waiting'
      }
    },
    statusText() {
      const status = this.status
      const fileStatusText = this.file.uploader.fileStatusText

      let txt = status
      if (typeof fileStatusText === 'function') {
        txt = fileStatusText(status, this.response)
      } else {
        txt = fileStatusText[status]
      }
      return txt || status
    },
    formatedTimeRemaining() {
      const timeRemaining = this.timeRemaining
      const file = this.file
      if (timeRemaining === Number.POSITIVE_INFINITY || timeRemaining === 0) {
        return ''
      }
      let parsedTimeRemaining = secondsToStr(timeRemaining)
      const parseTimeRemaining = file.uploader.opts.parseTimeRemaining
      if (parseTimeRemaining) {
        parsedTimeRemaining = parseTimeRemaining(timeRemaining, parsedTimeRemaining)
      }
      return parsedTimeRemaining
    }
  },
  watch: {
    status(newStatus, oldStatus) {
      if (oldStatus && newStatus === 'uploading' && oldStatus !== 'uploading') {
        this.tid = setTimeout(() => {
          this.progressingClass = 'uploader-file-progressing'
        }, 200)
      } else {
        clearTimeout(this.tid)
        this.progressingClass = ''
      }
    }
  },
  methods: {
    _actionCheck() {
      this.paused = this.file.paused
      this.error = this.file.error
      this.isUploading = this.file.isUploading()
    },
    pause() {
      this.file.pause()
      this._actionCheck()
      this._fileProgress()
    },
    resume() {
      this.file.resume()
      this._actionCheck()
    },
    remove() {
      this.file.cancel()
    },
    retry() {
      this.file.retry()
      this._actionCheck()
    },
    processResponse(message) {

      let res = message
      try {
        res = JSON.parse(message)
        // eslint-disable-next-line no-empty
      } catch (e) {

      }
      if (message == "") {

        res = { success: 200, message: "ok" }
      }
      this.response = res
    },
    fileEventsHandler(event, args) {
      const rootFile = args[0]
      const file = args[1]
      const target = this.list ? rootFile : file
      if (this.file === target) {
        if (this.list && event === 'fileSuccess') {
          this.processResponse(args[2])
          return
        }
        this[`_${event}`].apply(this, args)
      }
    },
    _fileProgress() {
      this.progress = this.file.progress()
      this.averageSpeed = this.file.averageSpeed
      this.currentSpeed = this.file.currentSpeed
      this.timeRemaining = this.file.timeRemaining()
      this.uploadedSize = this.file.sizeUploaded()
      this._actionCheck()
    },
    _fileSuccess(rootFile, file, message) {
      if (rootFile) {
        this.processResponse(message)
      }
      this._fileProgress()
      this.error = false
      this.isComplete = true
      this.isUploading = false
    },
    _fileComplete() {
      this._fileSuccess()
    },
    _fileError(rootFile, file, message) {
      this._fileProgress()
      this.processResponse(message)
      this.error = true
      this.isComplete = false
      this.isUploading = false
    }
  },
  mounted() {
    const staticProps = ['paused', 'error', 'averageSpeed', 'currentSpeed']
    const fnProps = [
      'isComplete',
      'isUploading',
      {
        key: 'size',
        fn: 'getSize'
      },
      {
        key: 'formatedSize',
        fn: 'getFormatSize'
      },
      {
        key: 'uploadedSize',
        fn: 'sizeUploaded'
      },
      'progress',
      'timeRemaining',
      {
        key: 'type',
        fn: 'getType'
      },
      {
        key: 'extension',
        fn: 'getExtension'
      }
    ]
    staticProps.forEach(prop => {
      this[prop] = this.file[prop]
    })
    fnProps.forEach((fnProp) => {
      if (typeof fnProp === 'string') {
        this[fnProp] = this.file[fnProp]()
      } else {
        this[fnProp.key] = this.file[fnProp.fn]()
      }
    })

    const handlers = this._handlers = {}
    const eventHandler = (event) => {
      handlers[event] = (...args) => {
        this.fileEventsHandler(event, args)
      }
      return handlers[event]
    }
    events.forEach((event) => {
      this.file.uploader.on(event, eventHandler(event))
    })
  },
  destroyed() {
    events.forEach((event) => {
      this.file.uploader.off(event, this._handlers[event])
    })
    this._handlers = null
  }
}
</script>

<style>
.uploader-file {
  position: relative;
  overflow: hidden;
}
</style>
