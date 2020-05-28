<script>
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import ListLabels from '@components/utils/list-labels'
import StartDueDates from '@components/projects/tasks/start-due-dates'
import BoardTaskOptions from '@components/projects/board-task-options'
import hexToRgba from 'hex-to-rgba'
import Types from '@components/projects/tasks/types'
import Efforts from '@components/projects/tasks/efforts'

export default {
  components: {
    ListUsers,
    StartDueDates,
    ListLabels,
    BoardTaskOptions,
    Types,
    Efforts
  },
  props: {
    workflow: {
      type: Object,
      required: false,
      default() {
        return []
      },
    },
    token: {
      type: String,
      required: true
    },
  },

  data() {
    return {
      loading: true,
      loadMoreLoading: false,
      showAddCard: false,
      showAddCardLoading: false,
      addTaskStatus: true,
      cardTitle: '',
      issueTitle: '',
      taskMember: '',
      taskMembers: [],
      totalTasks: 0,
      tasks: [],
      drag: false,
      objectTask: [],
      hideTasks: false,
      currentPage: 1,
      totalPages: 1,
    }
  },
  computed: {
    dragOptions() {
      return {
        animation: 0,
        group: 'tasks',
        disabled: false,
        ghostClass: 'ghost',
      }
    },
  },

  created() {
    this.getTasks()
  },
  methods: {
    
    getTasks() {
      if (this.currentPage <= this.totalPages) {

        Axios()
          .get(
            '/share-board/public/tasks/' + this.token + '/?workflow_id=' +
              this.workflow.id +
              '&page=' + this.currentPage
          )
          .then((response) => {
            
            let i = 0
            for(i in response.data.data) {
              this.tasks.push(response.data.data[i])
            }
            
            this.loading = false
            this.loadMoreLoading = false
            this.totalPages = response.data.total_pages
            this.totalTasks = response.data.total

            

          })
          .catch((e) => {
            console.error(e)
          })

      } else {
        this.loadMoreLoading = false
      }
    },

    hasImage(image) {
      return image !== false
    },

    isDraft(draft) {
      return draft
    },

    isBlocker(blocker) {
      return blocker
    },

    isBug(bug) {
      return bug
    },

    getUsers(users) {
      let strUsers = ''

      if (!users.length) {
        return ''
      }

      for (let i = 0; i < users.length; i++) {
        strUsers += users[i].username + ','
      }

      return strUsers.slice(0, -1)
    },

    getDate(date) {
      if (date === undefined) return ''

      return date.timezone
    },

    hasIcons(task) {
      return (
        this.isDraft(task.settings.is_draft) ||
        this.isBug(task.settings.is_bug) ||
        this.isBlocker(task.settings.is_blocker) ||
        task.stats.checklists ||
        task.stats.time_trackers ||
        task.stats.videos ||
        task.stats.comments ||
        task.stats.attachments ||
        task.stats.checklists
      )
    },

    loadMore: function() {
      if ( !this.loading ){
        this.currentPage += 1
        this.loadMoreLoading = true
        this.getTasks()        
      }
    },
    backgroundColor(hexColor) {
      return hexToRgba(hexColor, '0.06')
    },
  },

}
</script>

<template>
  <div
    :id="workflow.slug"
    class="board-workflow"
    :data-title="workflow.title"
    :data-slug="workflow.slug"
    :style="'border-top-color:' + workflow.color"
  >
    <header>
      <BoardTaskOptions :workflow="workflow" :total="totalTasks" :show-add-card="false"></BoardTaskOptions>
    </header>

    <div style="max-height: 100vh" >
      <ul
        v-infinite-scroll="loadMore" 
        infinite-scroll-disabled="busy" 
        infinite-scroll-distance="300"
        group="tasks"
        class="column list-group"
      >
        <span>
          <li
            v-for="task in tasks"
            :key="task.slug"
            :data-task="task.uuid"
            :data-username="getUsers(task.users)"
            class="list-group-item task-card">

            <div class="content-task-card" :style="(task.type) ? 'background: ' + backgroundColor(task.type.color) : ''">
              <span class="task-title">
                <strong v-if="task.code">{{ task.code }} - </strong>
                {{ task.title }}
              </span>

              <div v-if="hasImage(task.image)" class="cover-container">
                <img v-lazy="task.image" class="cover task-image" />
              </div>

              <div v-if="task.labels[0]">
                <ListLabels v-if="task.labels[0]" :labels="task.labels" :limit="15"></ListLabels>
              </div>

              <div v-if="task.type || task.effort" class="d-flex align-items-center flex-wrap">
                <Types v-if="task.type" :task="task"></Types>
                <Efforts :task="task"></Efforts>
              </div>

              <div
                v-if="task.users.length > 0"
                class="col-md d-flex justify-content-start"
                style="padding: 5px 0 5px 5px;"
              >
                <ListUsers size="26" :users="task.users" :link="true" :limit="6"></ListUsers>
              </div>

              <StartDueDates class="task-startDueDates" :task="task"></StartDueDates>

              <div v-if="hasIcons(task)" class="badge badge-light has-icons" style="padding-bottom: 0;">
                
                <div class="d-flex justify-content-between">
                  <div>
                    <span class="d-flex task-stats">
                      <small v-if="task.stats.checklists" v-b-tooltip.hover :title="$t('Checklist')">
                        <font-awesome-icon :icon="['far', 'list-ol']" />
                        {{ task.stats.checklists }}
                      </small>
                      <small v-if="task.stats.time_trackers" v-b-tooltip.hover :title="$t('Time Tracking')">
                        <font-awesome-icon :icon="['far', 'clock']" />
                        {{ task.stats.time_trackers }}
                      </small>
                      <small v-if="task.stats.videos" v-b-tooltip.hover :title="$t('Video')">
                        <font-awesome-icon :icon="['far', 'play-circle']" />
                        {{ task.stats.videos }}
                      </small>
                      <small v-if="task.stats.comments" v-b-tooltip.hover :title="$t('Comment')">
                        <font-awesome-icon :icon="['far', 'comments-alt']" />
                        {{ task.stats.comments }}
                      </small>
                      <small v-if="task.stats.attachments" v-b-tooltip.hover :title="$t('Attachment')">
                        <font-awesome-icon :icon="['far', 'paperclip']" />
                        {{ task.stats.attachments }}
                      </small>
                    </span>
                  </div>

                  <div class="task-icons">
                    <small v-if="isBug(task.settings.is_bug)" v-b-tooltip.hover :title="$t('Bug')">
                      <font-awesome-icon :icon="['far', 'bug']" />
                    </small>

                    <small v-if="isBlocker(task.settings.is_blocker)" v-b-tooltip.hover :title="$t('Blocker')">
                      <font-awesome-icon :icon="['far', 'shield-alt']" />
                    </small>

                    <small v-if="isDraft(task.settings.is_draft)" v-b-tooltip.hover :title="$t('Task Draft - Only you can see')">
                      <font-awesome-icon :icon="['far', 'eye']" />
                    </small>
                  </div>
                </div>
              </div>
            </div>

            <div
              v-if="task.stats.checklists && task.stats.checklist_percentage"
              class="task-progress"
              :data-width="task.stats.checklist_percentage"
            >
              <b-progress :max="100">
                <b-progress-bar
                  :value="task.stats.checklist_percentage"
                  :style="'background-color:' + workflow.color + ' !important'"
                >
                </b-progress-bar>
              </b-progress>
            </div>
          </li>
        </span>
      </ul>
      <div v-show="loadMoreLoading" class="text-center mt-2-px">
        <b-spinner tag="div" :label="$t('Loading')" small style="width: 12px;height: 12px;border-width: 2px;"></b-spinner>
      </div>
    </div>
    <div class="footer-column"></div>
  </div>
</template>

<style>
.content-task-card{
  cursor:default !important;
}

.board-task-options{
  display:none;
}

.board-task-options .board-task-icon{
  padding-left: 0 !important;
}

.task-title:hover{
  text-decoration: none !important;
  cursor:default !important;
}
</style>
