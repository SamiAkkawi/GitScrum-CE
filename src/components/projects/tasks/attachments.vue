<script>
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import Pagination from '@components/utils/pagination'
import { taskManager } from '@state/helpers'

export default {
  components: { ListUsers, Pagination },
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
      attachments: [],
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
      if ( data.item.name === 'attachment.reload' ){
        this.getAttachments(this.currentPage)
      }
    },
  },
  created() {
    this.getAttachments(this.currentPage)
  },
  methods: {
    checkMyTask() {
      let isMine = false

      if (this.task) {
        let currentUser = JSON.parse(localStorage.getItem('CURRENT_USER'))

        if (this.task.user.username === currentUser.username) {
          isMine = true
        } else {
          isMine = false
        }
      }

      return isMine
    },

    scrollToElem() {
      if (document.getElementById('task-attachments')) {
        let top = document.getElementById('task-attachments').offsetTop
        $('#b-modal-task').animate({ scrollTop: top }, 'slow')
      }
    },

    getAttachments(page) {
      this.scrollToElem()
      this.loading = true
      Axios()
        .get(
          'attachments/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&attachmentable_type=issues' +
            '&attachmentable_id=' +
            this.task.uuid +
            '&page=' +
            page
        )
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getAttachments(1)
            return
          }
          this.attachments = response.data.data
          if (response.data.data[0] && response.data.data[0].download && response.data.data[0].download.url) {
            this.task.image = response.data.data[0].download.url
          }
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loading = false
        })
        .catch((e) => {
          this.loading = false
        })
    },
    removeAttachment(attachment, index) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.attachments.splice(index, 1)
          Axios()
            .delete('attachments/' + attachment.id)
            .then((response) => {
              this.task.stats.attachments -= 1

              if (this.attachments[0] && this.attachments[0].mimetype.includes('image')) {
                this.task.image = this.attachments[0].download.url
              } else {
                this.task.image = null
              }
            })
        }
      })

    },
    isImage(attachment) {
      return attachment.mimetype.includes('image')
    },

    getIconByType(type) {
      let icon = ''
      switch (type) {
        case 'application/vnd.ms-excel':
        case 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet':
        case 'application/vnd.oasis.opendocument.spreadsheet':
          icon = '-excel'
          break
        case 'application/msword':
        case 'application/vnd.openxmlformats-officedocument.wordprocessing':
          icon = '-word'
          break
        case 'application/pdf':
          icon = '-pdf'
          break
        case 'text/plain':
          icon = '-alt'
          break
        default:
          icon = ''
      }

      return icon
    },
  },
}
</script>

<template>
  <b-row v-if="attachments[0]" class="mb-3">
    <b-col cols="1" class="task-left-icon">
      <font-awesome-icon :icon="['far', 'paperclip']" />
    </b-col>
    <b-col class="task-left-content">
      <h5 class="mb-3">{{ $t('Attachments') }}</h5>
      <silentbox-group>
        <div class="d-flex align-content-start flex-wrap">
          <div v-for="(attachment, index) in attachments" :key="attachment.id" class="gallery-box mb-10-px">
            <div v-if="isImage(attachment)">
              <div>
                <silentbox-item :src="attachment.download.url" :description="attachment.filename">
                  <div class="gallery-image">
                    <ListUsers :user="attachment.user" :link="true" size="22"></ListUsers>
                    <img :src="attachment.download.url" style="width:100%;" />
                  </div>
                </silentbox-item>
              </div>

              <div class="mt-1">
                <silentbox-item :src="attachment.download.url" :description="attachment.filename">
                  <span
                    class="font-weight-bold small"
                    :alt="attachment.filename"
                    :title="attachment.filename">
                    {{ attachment.filename | truncate(52) }}
                  </span>
                </silentbox-item>
                <div class="d-flex justify-content-between">
                  <span v-b-popover.hover.top=" attachment.created_at.date_for_humans" class="small">
                    {{ attachment.created_at.date_for_humans }}
                  </span>
                  <b-link 
                  v-if="authorize('tasks', 'update')" 
                  class="small text-danger" 
                  @click="removeAttachment(attachment, index)" 
                  v-text="$t('Delete')" />
                </div>
              </div>
            </div>

            <div v-if="!isImage(attachment)" class="wd-100p">
              
              <b-link :href="attachment.download.url" target="_blank">
                <div class="gallery-image">
                  <ListUsers :user="attachment.user" :link="true" size="22"></ListUsers>
                  <span class="icons d-block">
                    <font-awesome-icon
                      :icon="['fal', 'file' + getIconByType(attachment.mimetype)]"
                      class="text-secondary"
                      style="font-size:60px; top:-23px; position:relative;"
                    />
                  </span>
                </div>
              </b-link>

              <b-link :href="attachment.download.url" class="d-block mt-1" target="_blank">
                <span
                  class="font-weight-bold small"
                  :alt="attachment.filename"
                  :title="attachment.filename">
                  {{ attachment.filename | truncate(52) }}
                </span>
              </b-link>
              <div class="d-flex justify-content-between">
                <span v-b-popover.hover.top=" attachment.created_at.date_for_humans" class="small">
                  {{ attachment.created_at.date_for_humans }}
                </span>
                <b-link 
                v-if="authorize('tasks', 'update')" 
                class="small text-danger" 
                @click="removeAttachment(attachment, index)" 
                v-text="$t('Delete')" />
              </div>
            </div>
          </div>
        </div>
      </silentbox-group>
      <Pagination 
        :total-pages="totalPages" 
        :page="currentPage" 
        :total-rows="totalRows" 
        :per-page="perPage" 
        @change="getAttachments"></Pagination>
    </b-col>
  </b-row>
</template>
