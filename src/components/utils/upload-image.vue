<script>
import Axios from '@utils/axios'
import CropImage from '@components/utils/crop-image'

export default {
  components: {
    CropImage,
  },
  props: {
    btnTitle: {
      type: String,
      required: false,
      default: 'Upload'
    },
    btnRemoveTitle: {
      type: String,
      required: false,
      default: 'Remove'
    },
    btnClass: {
      type: String,
      required: false,
      default: '',
    },
    twoButtons: {
      type: Boolean,
      required: false,
      default: false,
    },
    image: {
      type: String,
      required: false,
    },
    size: {
      type: Number,
      required: false,
      default: 128
    },
    options: {
      type: Object,
      required: true,
    }
  },
  data() {
    return {
      loading: false,
      imageCropped: null,
      cropSize: this.options.cropSize ? this.options.cropSize : null,
      alertMessage: '',
      alertStatus: false,
      cropDimensions: this.options.cropSize ? ` (${this.options.cropSize.width} x ${this.options.cropSize.height})` : ''
    }
  },
  methods: {
    getImageCropped(imageCropped) {
      this.imageCropped = imageCropped
      this.resetFileInput()
      this.closeCropModal()
    },

    resetFileInput() {
      this.$refs.file.value = ''
    },

    onFileChange(e) {
      const file = e.target.files[0]
      this.imageCropped = URL.createObjectURL(file)
      this.callCropTool()
    },

    callCropTool() {
      this.$refs['modal'].show()
    },

    closeCropModal() {
      
      this.closeModal(this.$refs['modal'])
      if (this.imageCropped !== null) {
        this.uploadImage()
        this.$emit('action', this.imageCropped)
      }
    },

    deleteImage() {
      this.loading = true
      this.imageCropped = ''

      let params = {}
      params[this.options.paramName] = this.imageCropped

      Axios()
        .delete(this.options.url, params, {
          headers: this.options.headers,
        })
        .then((_) => {
          this.loading = false
          this.$emit('action', this.imageCropped)
        })
        .catch((error) => {
          alert(error.response.data.message)
        })
    },

    uploadImage() {
      this.loading = true
      let params = {}
      params[this.options.paramName] = this.imageCropped

      if (this.options.http === 'POST') {
        this.post(params)
      } else if (this.options.http === 'PUT') {
        this.put(params)
      }
    },

    post(params) {
      Axios()
        .post(this.options.url, params, {
          headers: this.options.headers,
        })
        .then((_) => {
          this.loading = false
          this.$emit('action', this.imageCropped)
        })
        .catch((error) => {
          alert(error.response.data.message)
        })
    },

    put(params) {
      Axios()
        .put(this.options.url, params, {
          headers: this.options.headers,
        })
        .then((_) => {
          this.loading = false
          this.$emit('action', this.imageCropped)
        })
        .catch((error) => {
          alert(error.response.data.message)
        })
    },
  },
}
</script>

<template>
  <b-overlay :show="loading" rounded="sm">
      
    <div class="mt-2" :style="'width:' + size + 'px !important'">
      <img v-show="image !== 'null'" :src="image" :style="'width:' + size + 'px !important'" class="rounded mb-2" />
    </div>

    <input ref="file" type="file" style="display: none" accept="image/*" @change="onFileChange" />
    
    <div v-if="!twoButtons">
      <button
        class="mb-1"
        :class="btnClass === '' ? 'btn btn-primary btn-sm' : btnClass"
        @click="$refs.file.click()">
        {{ btnTitle }}
      </button>
      <b-link v-if="image" class="d-block" @click="deleteImage">{{ btnRemoveTitle }}</b-link>
    </div>
    
    <div v-if="twoButtons">
      <button class="btn btn-primary mr-4-px p-0" @click="$refs.file.click()">
        <font-awesome-icon :icon="['fal', 'cloud-upload']" />
      </button>
      <b-button v-show="hasImage" class="btn btn-primary ml-4-px p-0" @click="deleteImage">
        <font-awesome-icon :icon="['far', 'times']" />
      </b-button>
    </div>
    
    <b-modal ref="modal" size="sm" hide-footer hide-header style="max-width: 500px !important;" @hide="resetFileInput">
      <template v-slot="{ hide }">
        <CropImage
          :img64="imageCropped"
          :round="false"
          :size="cropSize"
          @get-image-cropped="getImageCropped"
        ></CropImage>
      </template>
    </b-modal>
  </b-overlay>
</template>
