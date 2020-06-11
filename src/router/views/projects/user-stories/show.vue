<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import ListTasks from '@components/projects/tasks/list-tasks'
import TitleLoading from '@components/utils/title-loading'
import InputEditable from '@components/utils/input-editable' 
import TextareaEditable from '@components/utils/textarea-editable' 
import Pagination from '@components/utils/pagination'

export default {
  page: {
    title: 'User Story',
    meta: [{ name: '', content: '' }],
  },
  components: { Layout, ListUsers, ListTasks, TitleLoading, InputEditable, TextareaEditable, Pagination },
  data() {
    return {
      loading: true,
      tasks: [],
      userStory: [],
      userStoryPriorities: [],
      userStoryEndpoint: this.getUserStoryEndpoint(),
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
    }
  },
  mounted() {
    this.getUserStoryPriorities()
    this.getTasks(this.currentPage)
  },
  created() {
    this.getUserStory()
  },
  methods: {
    scrollToElem() {
      if (document.getElementById('task-assignees')) {
        let top = document.getElementById('task-assignees').offsetTop
        window.scroll({
          top: top,
          left: 0,
          behavior: 'smooth',
        })
      }
    },

    getUserStoryPriorities() {
      Axios()
        .get(
          'user-story-priorities/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          let data = response.data.data
          var arr = []

          for (let i = 0; i < data.length; i++) {
            arr.push({
              label: data[i].title,
              code: data[i].id,
              color: data[i].color,
            })
          }
          this.userStoryPriorities = arr
        })
        .catch((e) => {
          console.error(e)
        })
    },

    getUserStoryEndpoint() {
      return (
        'user-stories/' +
        this.$route.params.userStorySlug +
        '/?company_slug=' +
        this.$route.params.companySlug +
        '&project_slug=' +
        this.$route.params.projectSlug
      )
    },

    getUserStory() {
      Axios()
        .get(
          'user-stories/' +
            this.$route.params.userStorySlug +
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          this.userStory = response.data.data
          if( !this.userStory.priority ) {
            this.userStory.priority = {
              title: '',
              color: '#FFFFFF'
            }
          }

          // this.setSelectTextColor()
          this.loading = true
        })
        .catch((e) => {
          console.error(e)
        })
    },

    getTasks(page) {
      this.scrollToElem()
      this.loading = true
      Axios()
        .get(
          'tasks/?company_slug=' +
          this.$route.params.companySlug +
          '&project_slug=' +
          this.$route.params.projectSlug +
          '&user_story_slug=' +
          this.$route.params.userStorySlug +
          '&page=' +
          page
        )
        .then((response) => {
          this.tasks = response.data.data
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

    deleteUserStory() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete(this.getUserStoryEndpoint())
          .then((response) => {
            this.$router.push({
              name: 'projects.user-stories',
              params: {
                companySlug: this.$route.params.companySlug,
                projectSlug: this.$route.params.projectSlug,
              },
            })
          })
          .catch((e) => {
            console.error(e)
          })
        }
      })

    },

    update(params) {
      Axios()
        .put(
          'user-stories/' +
            this.$route.params.userStorySlug +
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug,
          params
        )
        .then((response) => {})
        .catch((e) => {
          this.getUserStory()
          console.error(e)
        })
    },

    updatePriority(priority) {
      this.userStory.priority.color = priority.color
      // this.setSelectTextColor()
      let params = {
        user_story_priority_id: priority.code,
      }
      this.update(params)
    },
    downloadExcel() {
      Axios()
        .get(
          '/export-time-trackings/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&user_story_slug=' +
            this.$route.params.userStorySlug,
          { responseType: 'blob' }
        )
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]))
          const link = document.createElement('a')
          link.href = url
          link.setAttribute('download', this.userStory.title + '.xlsx')
          document.body.appendChild(link)
          link.click()
        })
        .catch((e) => {
          console.error(e)
        })
    },
    titleUpdate(title){
      let params = {
        title: title,
      }
      this.update(params)
      this.userStory.title = title
    },

    updateAdditionalInformation(content){
      let params = { additional_information: content.text }
      this.update(params)
      this.sprint.additional_information = content.text
    },

    updateAcceptanceCriteria(content){
      let params = { acceptance_criteria: content.text }
      this.update(params)
      this.sprint.acceptance_criteria = content.text
    },
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
     <TitleLoading
        :title="$t('User Stories')"
        :subtitle="$t('User stories are short and simple descriptions of capabilities')"
        :loading="loading">
      </TitleLoading>
    </template>

    <div slot="content" class="user-story pt-10px">
      
      <b-container v-if="userStory">
        <b-row>
          <b-col>
            <b-card>

              <template v-slot:header>
                <div class="d-flex justify-content-between align-items-center">
                  <span v-text="$t('User Story')"></span>
                  <div class="d-flex align-items-center">
                    <router-link
                      :to="{
                      name: 'projects.board',
                      params: {
                        companySlug: $route.params.companySlug,
                        projectSlug: $route.params.projectSlug,
                      }, query: { userStorySlug: $route.params.userStorySlug } }" 
                      class="small"
                      v-text="$t('User Story in Board')">
                    </router-link>
                    <router-link
                      :to="{
                      name: 'projects.user-stories.assign-tasks',
                      params: {
                        companySlug: $route.params.companySlug,
                        projectSlug: $route.params.projectSlug,
                        userStorySlug: $route.params.userStorySlug } }"
                      class="ml-3 badge badge-success"
                      v-text="$t('Assign Tasks')">
                    </router-link>
                  </div>
                </div>
              </template>

              <InputEditable v-if="authorize('userStories', 'update')" 
                :placeholder="$t('Sprint Name')"
                :text="userStory.title"
                :current-object="userStory"
                @text-updated-blur="updateTitle"  
                @text-updated-enter="updateTitle"></InputEditable>
                <span v-else class="vlabeledit-label" v-text="userStory.title" />
                
                <div class="d-flex justify-content-between align-items-center">
                  <div class="small text-secondary ml-1">
                    {{ $t('Created on') }}
                    <span v-if="userStory.created_at" v-text="userStory.created_at.date_for_humans"></span> {{ $t('by') }} 
                    <router-link
                    v-if="userStory.user"
                    :to="{
                      name: 'profile.user',
                      params: { username: userStory.user.username } }"
                    v-text="userStory.user.name"></router-link>
                  </div>

                  <b-link
                    v-if="authorize('userStories', 'update')" 
                    class="small tx-uppercase"
                    :title="$t('Delete Sprint')"
                    @click="deleteUserStory">
                    <small class="small font-weight-bold">
                    <font-awesome-icon :icon="['far', 'trash']" class="mr-5-px" style="font-size:12px; color: #909CB8;" />
                    {{ $t('Delete User Story') }}
                    </small>
                  </b-link>
                </div>
            </b-card>
          </b-col>
        </b-row>
        <b-row v-if="userStory.stats" class="pt-2 pb-3">
          <b-col>
            <div class="d-flex align-items-center">
              <span class="txt-909CB8 percentage-text"> {{ parseFloat(userStory.stats.percentage) | percent(0) }}</span>
              <div class="progress wd-100 ml-10-px">
                <div
                  class="progress-bar"
                  role="progressbar"
                  :aria-valuenow="userStory.stats.percentage + 1"
                  aria-valuemin="0"
                  aria-valuemax="100"
                  :style="'width:' + userStory.stats.percentage + '%'">
                </div>
              </div>
            </div>
          </b-col>
        </b-row>
        <b-row>
          <b-col cols="3">
            
            <div style="position:sticky; top:115px">
              <b-jumbotron 
              v-if="userStory.stats" 
              :lead="userStory.stats.worked_hours" 
              class="sprint-jumbotron-yellow">
                <p><span class="d-block">{{ $t('Hours Worked') }}</span>
                  <b-link class="small" @click="downloadExcel">
                    {{ $t('Download TimeSheet') }}
                  </b-link></p>
              </b-jumbotron>
              <b-jumbotron 
              v-if="userStory.stats.story_points" 
              :lead="userStory.stats.story_points" 
              class="sprint-jumbotron-lime">
                <p v-text="$t('Effort')"></p>
              </b-jumbotron>
              <b-jumbotron 
              v-if="userStory.stats" 
              :lead="userStory.stats.closed_tasks + ' / ' + userStory.stats.total_tasks" 
              class="sprint-jumbotron-blue">
                <p v-text="$t('Tasks')"></p>
              </b-jumbotron>
              <b-card :header="$t('User Story Team')">
                <ListUsers
                  :link="true"
                  :users="userStory.users"
                  :limit="100"
                  :wrap="true"
                  class="list-users-left"></ListUsers>
              </b-card>

              <b-card class="mt-2">
                <div class="textarea-description mb-2">
                  <span class="small ml-1 font-weight-bold" v-text="$t('Additional information')" />
                  <TextareaEditable
                  v-if="authorize('userStories', 'update')" 
                  :placeholder="$t('User Story')"
                  :text="userStory.additional_information"
                  :current-object="sprint"
                  @text-updated-blur="updateAdditionalInformation"  
                  @text-updated-enter="updateAdditionalInformation"></TextareaEditable>
                  <span v-else class="vlabeledit-label" v-text="userStory.additional_information"></span>
                </div>
                <div class="textarea-description mb-2">
                  <span class="small ml-1 font-weight-bold" v-text="$t('Acceptance criteria')" />
                  <TextareaEditable
                  v-if="authorize('userStories', 'update')" 
                  :placeholder="$t('User Story')"
                  :text="userStory.acceptance_criteria"
                  :current-object="sprint"
                  @text-updated-blur="updateAcceptanceCriteria"  
                  @text-updated-enter="updateAcceptanceCriteria"></TextareaEditable>
                  <span v-else class="vlabeledit-label" v-text="userStory.acceptance_criteria"></span>
                </div>
              </b-card>
            </div>

            
          </b-col>
          <b-col cols="9">
            <b-card>
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
