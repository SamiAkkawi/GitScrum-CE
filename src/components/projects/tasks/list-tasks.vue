<script>
import { modalManager } from '@state/helpers'
import ListUsers from '@components/utils/list-users'
import Timer from '@components/projects/tasks/timer'

export default {
  components: { ListUsers, Timer },
  props: {
    items: {
      type: Array,
      required: false,
      default: null,
    },
    title: {
      type: String,
      required: false,
      default: null,
    },
    flag: {
      type: Boolean,
      required: false,
    },
    search: {
      type: Boolean,
      required: false,
    },
    modalFlag: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      loading: true
    }
  },
  watch: {
    items() {
      this.totalRows = this.items.length
    },
  },
  created() {
    this.loading = false
  },
  methods: {
    ...modalManager,

    modal(value, task) {
      if (this.modalFlag) {

        this.$router.push({
          name: 'projects.board.task-details',
          params: {
            companySlug: task.company.slug,
            projectSlug: task.project.slug,
            taskSlug: task.uuid,
          },
        })
        
      } else {
        this.open({ name: value, object: task })
      }
    },

    dueDate(date) {
      if (this.flag === true) {
        let currentDate = new Date()
        let taskDueDate = new Date(date)

        if (taskDueDate > currentDate) {
        } else {
          return 'color: red'
        }
      }
    },
    progressBar(item) {
      if (item.workflow) {
        return item.workflow.color
      }
      return ''
    },
  },
}
</script>

<template>
  <div>
    <div v-for="(item, index) in items" :key="index" class="list-tasks">
    
      <div class="d-flex align-items-start">

        <div class="mr-2">
          <ListUsers v-not-visible="'tablet'" :user="item.user" :link="true" size="28"></ListUsers>
        </div>
        
        <div style="width:100%">
          <div> 
            <b-link v-if="item.code" href="#" class="txt-primary-title" @click="modal('task', item)">{{ item.code }} - {{ item.title }}</b-link >
            <b-link v-if="!item.code" href="#" class="txt-primary-title" @click="modal('task', item)">{{ item.title }}</b-link>
            <div class="subtitle1 mt-1">
              <div>
                <router-link
                  v-b-tooltip.hover :title="item.project.name"
                  :to="{
                    name: 'projects.board',
                    params: {
                      companySlug: item.company.slug,
                      projectSlug: item.project.slug,
                    },
                  }" class="mr-1">{{ $t('Go to Project') }}
                </router-link> 

                <span>- {{ $t('Task created at') }} {{ item.created_at.date_for_humans }}</span>
                <span v-if="item.start_date.timezone" >
                    - {{ $t('Start Date') }}: {{ item.start_date.date_for_humans }}
                </span>
                <span v-if="item.start_date.timezone && item.due_date.timezone">
                  - 
                </span>
                <span v-if="item.due_date.timezone" :style="dueDate(item.due_date.timezone)">
                    {{ $t('Due Date') }}: {{ item.due_date.date_for_humans }}
                </span>
                 
              </div>
            </div>
            <div class="mt-1">
              <span
                v-if="item.workflow"
                class="badge badge-primary"
                :style="'color: ' + invertColor(item.workflow.color, true) + ';background:' + item.workflow.color"
              >
                {{ item.workflow.title }}
              </span>
              <span
                v-if="item.type"
                class="badge badge-primary"
                :style="'color: ' + invertColor(item.type.color, true) + ';background:' + item.type.color"
              >
                {{ item.type.title }}
              </span>
              <span v-if="item.effort" class="badge  badge-primary badge-light"> {{ item.effort.title }} </span>
              <Timer v-if="item.timer && authorize('tasks', 'read')" :task="item"></Timer>
            </div>
          </div>
        </div>
        
        <div class="ml-2">
          <ListUsers v-not-visible="'tablet'" :users="item.users" :link="true" :limit="2" size="28" :wrap="false"></ListUsers>
        </div>
      </div>

      <div v-show="item.stats.checklist_percentage" class="progress">
        <div
          class="progress-bar"
          :style="
            'background-color: ' + progressBar(item) + ' !important;width:' + item.stats.checklist_percentage + '%;'
          "
        ></div>
      </div>

      <hr />

    </div>
  </div>
</template>
