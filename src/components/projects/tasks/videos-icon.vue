<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { taskManager } from '@state/helpers'

export default {
  components: { ButtonLoading },
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
      youtubeURL: '',
      alertMessage: '',
      alertStatus: false,
    }
  },
  methods: {
    ...taskManager,

    addVideo() {
      this.loading = true
      this.alertStatus = false
      Axios()
        .post('videos/?company_slug=' + this.task.company.slug + '&project_slug=' + this.task.project.slug, {
          task_uuid: this.task.uuid,
          youtube_url: this.youtubeURL,
        })
        .then((response) => {
          this.youtubeURL = ''
          this.task.stats.videos += 1
          this.actionTask({ name: 'video.addList', object: response.data.data })
          this.loading = false
        })
    },
  },
}
</script>

<template>
<div>
  <b-button 
  v-if="authorize('tasks', 'create')" 
  v-b-toggle.videos-icon 
  class="btn btn-secondary btn-block"
  :style="(task.type) ? 'color: ' + 
  invertColor(task.type.color, true) + 
  ';background: ' + 
  opacityColor(task.type.color, '0.6') : ''"
  v-text="$t('Videos')"></b-button>
  <b-collapse id="videos-icon">
    <b-card>
      <b-input-group>
        <b-input-group-append class="wd-100">
          <b-form-input 
          v-model="youtubeURL" 
          :placeholder="$t('Copy the YouTube URL')"
          size="sm"></b-form-input>
          <ButtonLoading
          :loading="loading"
          type="btn-sm"
          icon="plus"
          @action="addVideo"></ButtonLoading>
        </b-input-group-append>
      </b-input-group>
    </b-card>
  </b-collapse>
</div>
</template>
