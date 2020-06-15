<script>
import Axios from '@utils/axios'
import { modalManager } from '@state/helpers'

import InputEditable from '@components/utils/input-editable' 
import TextareaEditable from '@components/utils/textarea-editable' 

import Assignees from '@components/projects/tasks/assignees'
import CustomFields from '@components/projects/tasks/custom-fields'
import Checklists from '@components/projects/tasks/checklists'
import ChecklistsIcon from '@components/projects/tasks/checklists-icon'
import Subtasks from '@components/projects/tasks/subtasks'
import SubtasksIcon from '@components/projects/tasks/subtasks-icon'
import MoveTasks from '@components/projects/tasks/move-tasks'

/*
import SubtasksConvertIcon from '@components/projects/tasks/subtasks-convert-icon'
*/
import PermanentLink from '@components/projects/tasks/permanent-link'
import Related from '@components/projects/tasks/related'
import ActivitiesIcon from '@components/projects/tasks/activities-icon'
import Attachments from '@components/projects/tasks/attachments'
import AttachmentsIcon from '@components/projects/tasks/attachments-icon'
import Videos from '@components/projects/tasks/videos'
import Types from '@components/projects/tasks/types'
import Efforts from '@components/projects/tasks/efforts'
import VideosIcon from '@components/projects/tasks/videos-icon'
import Comments from '@components/projects/tasks/comments'
import Timetracking from '@components/projects/tasks/timetracking'
import Timer from '@components/projects/tasks/timer'
import Labels from '@components/projects/tasks/labels'
import LabelsIcon from '@components/projects/tasks/labels-icon'
import SprintsIcon from '@components/projects/tasks/sprints-icon'
import UserStoriesIcon from '@components/projects/tasks/user-stories-icon'
import WorkflowsIcon from '@components/projects/tasks/workflows-icon'
import DatePicker from 'vue2-datepicker'
import moment from 'moment'

export default {
  components: {
    InputEditable,
    TextareaEditable,
    ActivitiesIcon,
    Assignees,
    CustomFields,
    Checklists,
    ChecklistsIcon,
    Subtasks,
    SubtasksIcon,
    PermanentLink,
    Related,
    Attachments,
    AttachmentsIcon,
    Videos,
    Types,
    Efforts,
    VideosIcon,
    Comments,
    Timetracking,
    Timer,
    Labels,
    LabelsIcon,
    MoveTasks,
    DatePicker,
    SprintsIcon,
    UserStoriesIcon,
    WorkflowsIcon,
  },
  data() {
    return {
      task: [],
      efforts: [],
      types: [],
      workflows: [],

      taskEffort: [],
      taskType: [],

      loading: true,
      editMode: false,
      modalShow: false,
      currentURL: '',

      refreshActivities: [],
      isClosed: false,
      txtComment: '',
      currentUser: JSON.parse(localStorage.getItem('CURRENT_USER'))
    }
  },
  computed: {
    ...modalManager,
  },
  watch: {
    statusModal(object) {
      
      if (object.item.name === 'task') {
        object.item.name = ''
        this.task = object.item.object
        this.init()
      }

    },
  },
  methods: {
    checkMyTask(task) {
      
      let isMine = false

      if (this.task) {
        
        if ( !this.currentUser ){
          isMine = false
        } else if (this.task.user.username === this.currentUser.username) {
          isMine = true
        } else {
          isMine = false
        }
      }
      return isMine
    },

    hideModal() {

      this.closeModal(this.$refs['modal'])
      
      if ( this.task.opened ) {
        location.href = this.currentURL
      } else {
        history.pushState(null, null, this.currentURL)
        
      }
      this.isClosed = true

    },

    init() {
      this.editMode = false
      this.loading = false
      this.modalShow = true
      this.isClosed = false
      this.pushStateOpen()
    },

    pushStateOpen() {
      
      if ( this.task.opened ) {
        this.currentURL = '/' + this.task.company.slug + '/' + this.task.project.slug
      } else {
        this.currentURL = this.$router.history.current.path
      }

      let urlModal = this.$router.resolve({
        name: 'projects.board.task-details',
        params: {
          companySlug: this.task.company.slug,
          projectSlug: this.task.project.slug,
          taskSlug: this.task.uuid,
        } }).href
        
      history.pushState(null, null, urlModal)
    },
    deleteTask() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete this task?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete(
            'tasks/' +
              this.task.uuid +
              '/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug
          )
          .then((response) => {
            this.hideModal()
            this.$eventBus.$emit('task-deleted', this.task)
            this.modalShow = false
          })
        }
      })

    },
    getWorkflows() {
      Axios()
        .get(
          'project-templates/workflow/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
        )
        .then((response) => {
          this.loading = false
          this.workflows = response.data.data
        })
    },
    editTask() {
      this.editMode = true
      this.getTaskEfforts()
      this.getTaskTypes()
      this.taskEffort = this.task.effort
      this.taskType = this.task.type
    },
    editTaskCancel() {
      this.editMode = false
    },
    editTaskUpdate() {
      Axios()
        .put(
          'tasks/' +
            this.task.uuid +
            '/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug,
          {
            title: this.task.title,
            description: this.task.description,
            config_issue_effort_id: this.taskEffort.id,
            config_issue_type_id: this.taskType.id,
          }
        )
        .then((response) => {
          this.task.title = this.task.title
          this.task.description = this.task.description
          this.task.effort = this.taskEffort
          this.task.type = this.taskType
          this.pushStateOpen()
        })
        .catch((e) => {})

      this.editMode = false
    },
    update(params) {
      Axios()
        .put(
          'tasks/' +
            this.task.uuid +
            '/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug,
          params
        )
        .then((response) => {
          if ( params.title ) {
            //this.task.slug = response.data.data.slug
            this.pushStateOpen()
          }
          return response
        })
        .catch((e) => {})
    },
    changeBlocked(evt) {
      this.update({ is_blocker: evt })
      this.task.settings.is_blocker = evt
    },
    changeDraft(evt) {
      this.update({ is_draft: evt })
      this.task.settings.is_draft = evt
    },
    changeBug(evt) {
      this.update({ is_bug: evt })
      this.task.settings.is_bug = evt
    },
    changeArchived(evt) {
      this.update({ is_archived: evt })
      this.task.settings.is_archived = evt
      this.$eventBus.$emit('reload-column', this.task)
    },
    changeStartDate(evt) {
      this.update({
        start_date: evt === null ? null : moment(String(evt)).format('YYYY-MM-DD HH:mm:ss'),
      })
      if (evt !== null) {
        this.task.start_date.timezone = moment(String(evt)).format('YYYY-MM-DD HH:mm:ss')
        this.task.start_date.timestamp = moment(String(evt)).format('X')
      } else {
        this.task.start_date.timezone = ''
        this.task.start_date.timestamp = ''
      }
    },
    changeDueDate(evt) {
      this.update({
        due_date: evt === null ? null : moment(String(evt)).format('YYYY-MM-DD HH:mm:ss'),
      })
      if (evt !== null) {
        this.task.due_date.timezone = moment(String(evt)).format('YYYY-MM-DD HH:mm:ss')
        this.task.due_date.timestamp = moment(String(evt)).format('X')
      } else {
        this.task.due_date.timezone = ''
        this.task.due_date.timestamp = ''
      }
    },
    updateTitle(content) {
      this.task.title = content.text
      this.update({ title: content.text })
    },
    removeSprint() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.update({ sprint_id: null })
          this.task.sprint = {}
          this.task.sprint.title = null
        }
      })

    },
    removeUserStory() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.update({ user_story_id: null })
        this.task.user_story = {}
        this.task.user_story.title = null
        }
      })
      
    },

    updateDescription(data) {
      this.task.description = data.text
      this.update({ description: data.text })
    },

    duplicateTask(){
      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to copy this task?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.hideModal()
          Axios()
            .post(
              'tasks/' +
                this.task.uuid +
                '/duplicate/?company_slug=' +
                this.task.company.slug +
                '&project_slug=' +
                this.task.project.slug
            )
            .then((response) => {
              this.$eventBus.$emit('reload-column', this.task)
              
            })
        }
      })
    },
  },
}
</script>

<template>
    <b-modal v-if="modalShow" id="b-modal-task" ref="modal" v-model="modalShow" size="lg" hide-footer hide-header>
      <b-container class="task-modal mb-4">
        <b-row>
          <b-col v-if="task.workflow" :style="'border-top: 15px solid ' + task.workflow.color" />
        </b-row>
        <b-row>
          <b-col cols="8" class="task-content-left">
            <b-row class="mt-3">
              <b-col cols="1"></b-col>
              <b-col>
                <div class="d-flex justify-content-between line-1">
                  <div class="d-flex align-items-center">
                    <WorkflowsIcon 
                      :task="task" 
                      :activities="refreshActivities" 
                      class="mr-2"></WorkflowsIcon>
                    <div class="mr-2">
                      <b-form-checkbox
                        v-model="task.settings.is_archived"
                        :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                        switch
                        class="mr-3" 
                        @input="changeArchived">
                        {{ $t('Archived') }}
                      </b-form-checkbox>
                    </div>
                  </div>
                  <div class="d-flex">
                    <Timer
                    v-if="task.timer"
                    :key="task.uuid"
                    :task="task"
                    :close="isClosed"
                    scope="modal"></Timer>
                    <b-dropdown v-if="authorize('tasks', 'update', checkMyTask(task))" right class="styled-dropdown line-1-dropdown">
                      <template v-slot:button-content>
                        <font-awesome-icon :icon="['far', 'ellipsis-h']" />
                      </template>
                      <b-dropdown-item @click="duplicateTask">
                        <font-awesome-icon :icon="['far', 'clone']"/>
                        {{ $t('Copy Task') }}
                      </b-dropdown-item>
                      <b-dropdown-item @click="deleteTask">
                        <font-awesome-icon :icon="['far', 'trash']"/>
                        {{ $t('Delete') }}
                      </b-dropdown-item>
                    </b-dropdown>
                  </div>
                </div>
              </b-col>
            </b-row>
            <hr>
            <b-row>
              <b-col cols="1">
                <font-awesome-icon
                  :icon="['far', 'align-justify']"
                  class="task-icon" />
              </b-col>
              <b-col>
                <div class="input-title">
                  <InputEditable 
                  v-if="authorize('tasks', 'update', checkMyTask(task))" 
                  :placeholder="$t('Task Title')"
                  :text="task.title"
                  :current-object="task"
                  @text-updated-blur="updateTitle"  
                  @text-updated-enter="updateTitle" />
                  <span v-else class="vlabeledit-label" v-text="task.title" />
                </div>
                <div class="small">
                  <router-link
                  :to="{
                  name: 'projects.board',
                  params: {
                    companySlug: this.$route.params.companySlug,
                    projectSlug: this.$route.params.projectSlug } }"
                  v-text="task.project.name" />
                   - 
                  {{ $t('Task created by') }}
                  <router-link
                  :to="{
                  name: 'profile.user',
                  params: { username: task.user.username } }"
                  v-text="task.user.name" />
                  {{ $t('at') }}
                  <span
                  v-b-popover.hover.top="momentLocale(task.created_at.timestamp)"
                  v-text="task.created_at.date_for_humans" />
                </div>
              </b-col>
            </b-row>
            <b-row class="mt-3">
              <b-col cols="1"></b-col>
              <b-col>
                <div class="d-flex">
                  <Types :task="task" :dropdown="true"></Types>
                  <Efforts :task="task" :dropdown="true"></Efforts>
                </div>
              </b-col>
            </b-row>
            <b-row class="mt-3">
              <b-col cols="1">
                <font-awesome-icon
                  :icon="['far', 'users']"
                  class="task-icon mt-1" />
              </b-col>
              <b-col class="d-flex">
                <Assignees :task="task" :activities="refreshActivities" :wrap="true" class="col-md-6 pl-0"></Assignees>
                <Labels :task="task" class="mb-1 ml-3 pr-2 col-md-6 text-right"></Labels>
              </b-col>
            </b-row>
            <hr>
            <b-row class="mb-3">
              <b-col cols="1" class="task-left-icon">
                <font-awesome-icon
                  :icon="['far', 'calendar-alt']"
                  class="task-icon" />
              </b-col>
              <b-col class="d-flex modal-task-date task-left-content">
                <div class="modal-task-dates">
                  <div class="start_date_content">{{ $t('Start date') }}</div>
                  <DatePicker
                    v-model="task.start_date.timezone"
                    lang="en"
                    type="datetime"
                    format="YYYY-MM-DD hh:mm A"
                    :time-picker-options="{
                      start: '00:00', step: '00:30', end: '23:30' }"
                    confirm
                    class="start_date_input"
                    :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                    :not-after="task.due_date.timezone"
                    @change="changeStartDate"></DatePicker>
                </div>
                <div class="modal-task-dates ml-2">
                  <div class="due_date_content">{{ $t('Due date') }}</div>
                  <DatePicker
                    v-model="task.due_date.timezone"
                    lang="en"
                    type="datetime"
                    format="YYYY-MM-DD hh:mm A"
                    :time-picker-options="{
                      start: '00:00', step: '00:30', end: '23:30' }"
                    confirm
                    class="due_date_input"
                    :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                    :not-before="task.start_date.timezone"
                    @change="changeDueDate"></DatePicker>
                </div>
              </b-col>
            </b-row>
            <b-row class="mb-3">
              <b-col cols="1" class="task-left-icon">
                <font-awesome-icon 
                :icon="['far', 'align-right']" 
                class="task-icon" />
              </b-col>
              <b-col class="textarea-description task-left-content">
                <TextareaEditable
                  v-if="authorize('tasks', 'update', checkMyTask(task)) || 
                    (task.description && task.description.trim() !== '')"
                  :placeholder="$t('Task Description')"
                  :text="task.description"
                  :current-object="task"
                  class="wd-100"
                  @text-updated-blur="updateDescription"  
                  @text-updated-enter="updateDescription"></TextareaEditable>
                  <span v-else class="vlabeledit-label" v-text="task.description"></span>
              </b-col>
            </b-row>
            <b-row v-if="task.sprint.title || task.user_story.title" class="mb-3">
              <b-col cols="1"></b-col>
              <b-col style="padding-left: 8px;">
                  <div v-if="task.sprint.title !== null" class="badge badge-light mb-1">
                    {{ $tc('Sprint', 1) }}:&nbsp;
                    <strong>
                      <router-link
                      :to="{
                      name: 'projects.sprints.show',
                      params: {
                      companySlug: task.company.slug,
                      projectSlug: task.project.slug,
                      sprintSlug: task.sprint.slug } }"
                      class="txt-link txt-primary"
                      :alt="task.sprint.title"
                      :title="task.sprint.title">
                      {{ task.sprint.title }}
                      </router-link>
                      <span v-if="authorize('tasks', 'update', checkMyTask(task))" class="txt-link mr-0-px ml-10-px" @click="removeSprint">
                        <font-awesome-icon :icon="['far', 'times']" style="font-size:14px" class="txt-3D4F9F" />
                      </span>
                    </strong>
                  </div>
                  <div v-if="task.user_story.title !== null" class="badge badge-light">
                      {{ $tc('User Story', 1) }}:&nbsp;
                      <strong>
                        <router-link
                        :to="{
                        name: 'projects.user-stories.show',
                        params: {
                        companySlug: task.company.slug,
                        projectSlug: task.project.slug,
                        userStorySlug: task.user_story.slug } }"
                        class="txt-link txt-primary"
                        :alt="task.user_story.title"
                        :title="task.user_story.title">
                        {{ task.user_story.title | truncate(55) }}
                        </router-link>
                        <span v-if="authorize('tasks', 'update', checkMyTask(task))" class="txt-link mr-0-px ml-10-px" @click="removeUserStory">
                          <font-awesome-icon :icon="['far', 'times']" style="font-size:14px" class="txt-3D4F9F" />
                        </span>
                      </strong>
                  </div>
              </b-col>
            </b-row>
            <Checklists :task="task"></Checklists>
            <CustomFields :task="task"></CustomFields>
            <Attachments :task="task"></Attachments>
            <Videos :task="task"></Videos>
            <Related
            :task="task"
            :modal-flag="true"></Related>
            <Subtasks
            :task="task"
            :modal-flag="true" />
            <Comments :task="task"></Comments>
          </b-col>
          <b-col cols="4" :style="(task.type) ? 'background: ' + opacityColor(task.type.color, '0.2') : ''">
            <div class="d-flex justify-content-between mb-1">
                <div class="d-flex align-items-center pl-0 mt-4">
                  <b-form-checkbox
                    v-model="task.settings.is_blocker"
                    :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                    switch
                    class="mr-3" 
                    @input="changeBlocked">
                    {{ $t('Blocked') }}
                  </b-form-checkbox>
                  <b-form-checkbox
                    v-model="task.settings.is_draft"
                    :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                    switch
                    class="mr-3" 
                    @input="changeDraft">
                    {{ $t('Draft') }}
                  </b-form-checkbox>
                  <b-form-checkbox
                    v-model="task.settings.is_bug"
                    :disabled="!authorize('tasks', 'update', checkMyTask(task))"
                    switch
                    @input="changeBug">
                    {{ $t('Bug') }}
                  </b-form-checkbox>
                </div>
                <b-link v-if="authorize('tasks', 'read')" class="task-close" @click="hideModal">
                  <font-awesome-icon :icon="['fas', 'times']" />
                </b-link>
            </div>
            <div class="task-actions">
              <ActivitiesIcon :task="task" context="issue"></ActivitiesIcon>
              <Timetracking v-if="task.timer" :task="task"></Timetracking>
              <ChecklistsIcon :task="task"></ChecklistsIcon>
              <SubtasksIcon :task="task"></SubtasksIcon>
              <AttachmentsIcon :task="task"></AttachmentsIcon>
              <VideosIcon :task="task"></VideosIcon>
              <LabelsIcon :task="task"></LabelsIcon>
              <SprintsIcon v-if="task.has_sprints" :task="task" :activities="refreshActivities"></SprintsIcon>
              <UserStoriesIcon v-if="task.has_user_stories" :task="task" :activities="refreshActivities"></UserStoriesIcon>
              
              <PermanentLink :task="task" class="mt-4"></PermanentLink>
              <MoveTasks :task="task"></MoveTasks>
            </div>
          </b-col>
        </b-row>
      </b-container>
    </b-modal>
</template>