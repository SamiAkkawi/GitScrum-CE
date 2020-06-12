<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import ListTasks from '@components/projects/tasks/list-tasks'
import TitleLoading from '@components/utils/title-loading'
import Burndown from '@components/projects/sprints/burndown'
import DatePicker from 'vue2-datepicker'
import InputEditable from '@components/utils/input-editable' 
import TextareaEditable from '@components/utils/textarea-editable' 
import Pagination from '@components/utils/pagination'

export default {
  page: {
    title: 'Sprint',
    meta: [{ name: '', content: '' }],
  },
  components: {
    Layout,
    ListUsers,
    ListTasks,
    Burndown,
    DatePicker,
    InputEditable,
    TextareaEditable,
    TitleLoading,
    Pagination
  },
  data() {
    return {
      loading: false,
      sprint: null,
      tasks: [],
      sprintDates: [],
      sprintStatus: '',
      sprintStatuses: [],
      sprintEndpoint: this.getSprintEndpoint(),
      timeboxEditMode: false,
      timeboxChangesLoading: false,
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
    }
  },
  mounted() {
    this.getSprint()
    this.getSprintStatuses()
    this.getTasks(this.currentPage)
  },
  methods: {
    
    getSprintStatuses() {
      Axios()
        .get('sprints/statuses?company_slug=' + this.$route.params.companySlug)
        .then((response) => {
          this.sprintStatuses = response.data.data
        })
    },

    getSprintEndpoint() {
      return (
        'sprints/' +
        this.$route.params.sprintSlug +
        '/?company_slug=' +
        this.$route.params.companySlug +
        '&project_slug=' +
        this.$route.params.projectSlug
      )
    },

    getSprint() {
      this.loading = true
      Axios()
        .get('sprints/' + this.$route.params.sprintSlug + 
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug)
        .then((response) => {
          this.sprint = response.data.data
          this.sprintStatus = this.sprint.status.slug
          document.title = this.sprint.title

          if (this.sprint.date_start && this.sprint.date_finish) {
            this.sprintDates[0] = new Date(this.sprint.date_start)
            this.sprintDates[1] = new Date(this.sprint.date_finish)
          }

          if (this.timeboxChangesLoading) {
            this.handleDatePicker()
          }

          this.timeboxChangesLoading = false
          this.loading = false
        })
    },

    getTasks(page) {
      this.gotoScrollTop();
      this.loading = true
      Axios()
        .get(
          'tasks/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&sprint_slug=' +
            this.$route.params.sprintSlug +
            '&page=' +
            page
        )
        .then((response) => {
          this.loading = false
          this.tasks = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
        })
    },

    deleteSprint() {
        
      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        if(value){
          Axios()
          .delete(this.getSprintEndpoint())
          .then((response) => {
            this.$router.push({
              name: 'projects.sprints',
              params: {
                companySlug: this.$route.params.companySlug,
                projectSlug: this.$route.params.projectSlug,
              },
            })
          })
        }
      })

    },

    update(params) {
      Axios()
        .put('sprints/' + this.$route.params.sprintSlug + 
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug, params)
        .then((response) => {
          if (this.timeboxChangesLoading) {
            this.getSprint()
          }
        })
        .catch((e) => {
          console.error(e)
        })
    },

    changeDate(date) {
      this.timeboxChangesLoading = true

      let dateStartFormated = null
      let dateFinishFormated = null

      if (this.sprintDates) {
        if (this.sprintDates[0]) {
          let dateStart = new Date(this.sprintDates[0])
          dateStartFormated = dateStart.getFullYear() + '-' + (dateStart.getMonth() + 1) + '-' + dateStart.getDate()
        }

        if (this.sprintDates[1]) {
          let dateFinish = new Date(this.sprintDates[1])
          dateFinishFormated = dateFinish.getFullYear() + '-' + (dateFinish.getMonth() + 1) + '-' + dateFinish.getDate()
        }
      }

      let params = {
        date_start: dateStartFormated,
        date_finish: dateFinishFormated,
      }

      this.update(params)
    },

    handleDatePicker() {
      this.timeboxEditMode = !this.timeboxEditMode
      this.$refs.datepick.popupVisible = this.timeboxEditMode
    },

    downloadExcel() {
      Axios()
        .get(
          '/export-time-trackings/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&sprint_slug=' +
            this.$route.params.sprintSlug,
          { responseType: 'blob' }
        )
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]))
          const link = document.createElement('a')
          link.href = url
          link.setAttribute('download', this.sprint.title + '.xlsx')
          document.body.appendChild(link)
          link.click()
        })
    },

    updateTitle(title){
      let params = { title: title.text }
      this.update(params)
      this.sprint.title = title.text
    },

    updateDescription(description){
      let params = { description: description.text }
      this.update(params)
      this.sprint.description = description.text
    },

    updateStatus(status) {
      let params = {sprint_status_slug: status,}
      this.update(params)
    },

  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
     <TitleLoading
        :title="$t('Sprints')"
        :subtitle="$t('Sprint is one timeboxed iteration of a continuous cycle')"
        :loading="loading">
      </TitleLoading>
    </template>

    <div slot="content" class="sprint pt-10px">
      
      <b-container v-if="sprint">
        <b-row>
          <b-col>
            <b-card>
              <template v-slot:header>
                <div class="d-flex justify-content-between align-items-center">
                  <span v-text="$t('Sprint')"></span>
                  <div class="d-flex align-items-center">
                    <router-link
                      v-if="authorize('sprints', 'update')"
                      :to="{
                      name: 'projects.board',
                      params: {
                        companySlug: $route.params.companySlug,
                        projectSlug: $route.params.projectSlug,
                      }, query: { sprintSlug: $route.params.sprintSlug } }"
                    class="small">
                    {{ $t('Sprint in Board') }}</router-link>
                    <router-link
                      v-if="authorize('sprints', 'update')"
                      :to="{
                      name: 'projects.sprints.assign-tasks',
                      params: {
                        companySlug: $route.params.companySlug,
                        projectSlug: $route.params.projectSlug,
                        sprintSlug: $route.params.sprintSlug } }"
                      class="ml-3 badge badge-success">
                      {{ $t('Assign Tasks') }}</router-link>
                  </div>
                </div>
              </template>
              <div class="title-component">
                <InputEditable v-if="authorize('sprints', 'update')" 
                :placeholder="$t('Sprint Name')"
                :text="sprint.title"
                :current-object="sprint"
                @text-updated-blur="updateTitle"  
                @text-updated-enter="updateTitle"></InputEditable>
                <span v-else class="vlabeledit-label" v-text="sprint.title"></span>
                <div class="textarea-description mb-2">
                  <TextareaEditable
                  v-if="authorize('sprints', 'update')" 
                  :placeholder="$t('Sprint Name')"
                  :text="sprint.description"
                  :current-object="sprint"
                  @text-updated-blur="updateDescription"  
                  @text-updated-enter="updateDescription"></TextareaEditable>
                  <span v-else class="vlabeledit-label" v-text="sprint.description"></span>
                </div>
                <div class="d-flex justify-content-between align-items-center">
                  <span class="small text-secondary ml-1">
                    {{ $t('Created on') }}
                    <span v-if="sprint.created_at" v-text="sprint.created_at.date_for_humans"></span> 
                    {{ $t('by') }} 
                    <router-link
                    v-if="sprint.user"
                    :to="{
                      name: 'profile.user',
                      params: { username: sprint.user.username } }"
                    v-text="sprint.user.name"></router-link>
                  </span>
                  <b-link
                    v-if="authorize('userStories', 'update')" 
                    class="small tx-uppercase"
                    :title="$t('Delete Sprint')"
                    @click="deleteSprint">
                    <small class="small font-weight-bold">
                    <font-awesome-icon :icon="['far', 'trash']" class="mr-5-px" style="font-size:12px; color: #909CB8;" />
                    {{ $t('Delete Sprint') }}
                    </small>
                  </b-link>
                </div>
              </div>
              <div class="d-flex justify-content-between align-items-center">
                <div class="d-flex justify-content-start" style="height: 25px;">
                  <span class="font-weight-bold">{{ $t('Timebox') }}</span>
                  <div v-if="sprint" class="sprint-timebox ml-10-px wd-150">
                    <div :class="timeboxEditMode ? 'none' : ''">
                      <span v-text="sprint.timebox"></span>
                      <font-awesome-icon
                        v-if="authorize('sprints', 'update')"
                        :icon="['far', 'calendar-alt']"
                        class="calendar-edit ml-2"
                        @click="handleDatePicker"
                      />
                    </div>
                    <div :class="timeboxEditMode ? 'd-flex timebox-datepicker' : 'none'">
                      <DatePicker
                        ref="datepick"
                        v-model="sprintDates"
                        range
                        lang="en"
                        format="YYYY-MM-DD"
                        confirm
                        :disabled="timeboxChangesLoading"
                        @change="changeDate"
                        @blur="handleDatePicker"
                      ></DatePicker>
                      <b-spinner
                        v-show="timeboxChangesLoading"
                        :label="$t('Loading')"
                        variant="secondary"
                        small
                        class="title-loading m-7-px"
                      ></b-spinner>
                    </div>
                  </div>
                  <div v-if="sprint">
                    <span class="font-weight-bold ml-4">{{ $t('Duration') }}</span>
                    <span class="ml-3" v-text="sprint.duration"></span>
                  </div>
                </div>
                <div class="d-flex">
                  <b-form-select 
                  v-model="sprintStatus" 
                  :options="sprintStatuses" 
                  value-field="slug" 
                  text-field="title"
                  size="sm"
                  @change="updateStatus"></b-form-select>
                </div>
              </div>
            </b-card>
          </b-col>
        </b-row>
        <b-row v-if="sprint.stats" class="pt-2 pb-3">
          <b-col>
            <div class="d-flex align-items-center">
              <span class="txt-909CB8 percentage-text"> {{ parseFloat(sprint.stats.percentage) | percent(0) }}</span>
              <div class="progress wd-100 ml-10-px">
                <div
                  class="progress-bar"
                  role="progressbar"
                  :aria-valuenow="sprint.stats.percentage + 1"
                  aria-valuemin="0"
                  aria-valuemax="100"
                  :style="'width:' + sprint.stats.percentage + '%'">
                </div>
              </div>
            </div>
          </b-col>
        </b-row>
        <b-row>
          <b-col cols="3">
            <div style="position:sticky; top:115px">
              <b-jumbotron 
              v-if="sprint.stats" 
              :lead="sprint.stats.worked_hours" 
              class="sprint-jumbotron-yellow">
                <p><span class="d-block">{{ $t('Hours Worked') }}</span>
                  <b-link class="small" @click="downloadExcel">
                    {{ $t('Download TimeSheet') }}
                  </b-link></p>
              </b-jumbotron>
              <b-jumbotron 
              v-if="sprint.stats.story_points" 
              :lead="sprint.stats.story_points" 
              class="sprint-jumbotron-lime">
                <p v-text="$t('Effort')"></p>
              </b-jumbotron>
              <b-jumbotron 
              v-if="sprint.stats" 
              :lead="sprint.stats.closed_tasks + ' / ' + sprint.stats.total_tasks" 
              class="sprint-jumbotron-blue">
                <p v-text="$t('Tasks')"></p>
              </b-jumbotron>
              <b-card v-if="sprint.users.length" :header="$t('Sprint Team')">
                <ListUsers
                  :link="true"
                  :users="sprint.users"
                  :limit="100"
                  :wrap="true"
                  class="list-users-left"></ListUsers>
              </b-card>
            </div>
          </b-col>
          <b-col cols="9">
            <b-card :header="$t('Sprint Burndown')" class="sprint-burndown">
              <b-tabs class="tabs-burndown" style="margin-left: -8px;">
                <b-tab :title="$t('Burndown by Task')" active>
                  <Burndown v-if="sprint.charts" type-name="Task" :data="sprint.charts.burndown_tasks"></Burndown>
                </b-tab>
                <b-tab :title="$t('Burndown by Effort')">
                  <Burndown v-if="sprint.charts" type-name="Effort" :data="sprint.charts.burndown_efforts"></Burndown>
                </b-tab>
              </b-tabs>
            </b-card>

            <b-card v-if="tasks.length" class="mt-2">
              <ListTasks class="pl-2 pr-3" :items="tasks" :search="true" title="" :flag="true"></ListTasks>
            </b-card>

            <Pagination 
              :total-pages="totalPages" 
              :page="currentPage" 
              :total-rows="totalRows" 
              :per-page="perPage" 
              @change="getTasks"></Pagination>

          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>
