<script>
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import { taskManager } from '@state/helpers'

export default {
  components: { ListUsers },
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
      videos: [],
      loading: true,
      visible: true,
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
    }
  },
  computed: {
    ...taskManager,
  },
  watch: {
    statusTask(data){
      if ( data.item.name === 'video.addList' ){
        this.getVideos()
      }
    },
  },
  created() {
    this.getVideos(this.currentPage)
  },
  methods: {
    getVideos(page) {
      this.loading = true
      Axios()
        .get(
          'videos/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid +
            '&page=' +
            page
        )
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getVideos(1)
            return
          }
          this.videos = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },
    removeVideo(video, index) {
      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        if(value){
          this.videos.splice(index, 1)
          Axios()
          .delete('videos/' + video.id)
          .then((response) => {
            this.task.stats.video -= 1
          })
        }
      })
    },
  },
}
</script>

<template>
  <b-row v-if="videos[0]" class="mb-3">
    <b-col cols="1" class="task-left-icon">
      <font-awesome-icon :icon="['far', 'video']" />
    </b-col>
    <b-col class="task-left-content">
      <h5 class="mb-3">{{ $t('Videos') }}</h5>
      <silentbox-group>
        <div class="d-flex align-content-start flex-wrap">
          <div v-for="(video, index) in videos" :key="index" class="gallery-box">
            <silentbox-item :src="video.url" :description="video.title">
              <div class="gallery-image">
                <ListUsers :user="video.user" :link="true" size="22"></ListUsers>
                <img :src="video.thumbnail" style="width:100%; position:relative; top:-50px" class="shadow-sm" />
              </div>
            </silentbox-item>
            <div class="mt-5-px">
              <silentbox-item :src="video.url" :description="video.title">
                <b-link
                  class="small"
                  :alt="video.title"
                  :title="video.title" >
                  {{ video.title | truncate(52) }}
                </b-link>
              </silentbox-item>
              <div class="d-flex justify-content-between">
                <span v-b-popover.hover.top="video.created_at.timezone" class="small">
                  {{ video.created_at.date_for_humans }}
                </span>
                <b-link 
                v-if="authorize('tasks', 'update')" 
                class="small text-danger" 
                @click="removeVideo(video, index)" 
                v-text="$t('Delete')" />
              </div>
            </div>
          </div>
        </div>
      </silentbox-group>
      <div v-if="totalPages > 1" class="d-flex justify-content-center mg-b-30">
        <b-pagination
          v-model="currentPage"
          hide-goto-end-buttons
          class="paginator"
          :total-rows="totalRows"
          :per-page="perPage"
          @change="getVideos">
          <template slot="prev-text">
            <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
          </template>
          <template slot="next-text">
            <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
          </template>
        </b-pagination>
      </div>
    </b-col>
  </b-row>
</template>
