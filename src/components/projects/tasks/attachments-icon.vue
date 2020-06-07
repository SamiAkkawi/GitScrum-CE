<script>
import appConfig from '@src/app.config'
import vue2Dropzone from 'vue2-dropzone'
import 'vue2-dropzone/dist/vue2Dropzone.min.css'
import DropboxPicker from '@components/utils/dropbox-picker'
import GDrivePicker from '@components/utils/gdrive-picker'
import TitleLoading from '@components/utils/title-loading'
import { taskManager } from '@state/helpers'

export default {
  components: { vueDropzone: vue2Dropzone, DropboxPicker, GDrivePicker, TitleLoading },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
  },
  data() {
    return {
      loading: false,

      dropzoneOptions: {
        url:
          appConfig.APIBaseURL +
          'attachments/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&attachmentable_type=issues' +
          '&attachmentable_id=' +
          this.task.uuid,
        thumbnailWidth: 50,
        thumbnailHeight: 50,
        maxFilesize: 100,
        headers: { Authorization: localStorage.getItem('ACCESS_TOKEN') },
        paramName: 'file',
      },
      uploadFilesCount: 0,
      showPicker: true,
    }
  },
  created(){
    this.whois()
  },
  methods: {
    ...taskManager,

    whois(){

      if ( window.location.hostname !== 'localhost:8080' || window.location.hostname !== 'gitscrum.com' ){
        this.showPicker = false;
      }

    },
    afterComplete(file) {
      file.previewElement.parentNode.removeChild(file.previewElement)
      this.uploadFilesCount--
      this.handleDropzonePreview()
      this.actionTask({ name: 'attachment.reload' })
      this.loading = false
    },

    updateImage() {
      this.refresh.push([])
      this.activities.push([])
    },

    sending() {
      this.uploadFilesCount++
      this.loading = true
      this.handleDropzonePreview()
    },

    handleDropzonePreview() {
      let dropzoneMessage = document.getElementsByClassName('dropzone-custom-content')
      dropzoneMessage[0].parentNode.style.display = this.uploadFilesCount == 0 ? 'block' : 'none'
    },
  },
}
</script>

<template>
  <div>
    <button 
      v-if="authorize('tasks', 'create')" 
      v-b-toggle.attachments-icon
      class="btn btn-secondary btn-block">
      {{ $t('Attachments') }}
    </button>
    <b-collapse id="attachments-icon">
      <b-card>
        <vue-dropzone
          id="attachments-icon-dropzone"
          :options="dropzoneOptions"
          :use-custom-slot="true"
          @vdropzone-file-added="sending"
          @vdropzone-complete="afterComplete($event)">
          <h3 class="h6 font-weight-bold">{{ $t('Drag and drop to upload') }}</h3>
          <div class="small">...{{ $t('or click to select a file from your computer') }}</div>
        </vue-dropzone>
        <div v-if="showPicker" class="mt-2">
          <DropboxPicker
            :id="task.uuid"
            :company-slug="task.company.slug"
            :project-slug="task.project.slug"
            attachmentable-type="issues"
            @image-uploaded="updateImage"></DropboxPicker>
          <GDrivePicker
            :id="task.uuid"
            :company-slug="$route.params.companySlug"
            :project-slug="$route.params.projectSlug"
            attachmentable-type="issues"
            class="mt-2"
            @image-uploaded="updateImage"></GDrivePicker>
        </div>
      </b-card>
    </b-collapse>
  </div>
</template>
