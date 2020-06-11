<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import ListTasks from '@components/projects/tasks/list-tasks'
import Pagination from '@components/utils/pagination'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'User Story Assign Tasks',
    meta: [{ name: '', content: '' }],
  },
  components: { Layout, TitleLoading, Pagination, ListTasks, ButtonLoading },
  data() {
    return {
      loading: true,
      assignLoading: false,
      btnLoading: false,
      tasks: [],
      items: [],
      currentItem: '',
      data: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      filtered: [],
      filterTitle: '',
      assignedTasks: [],
      userStoryPriorities: [],
      userStoryEndpoint: this.getUserStoryEndpoint(),
      fields: [
        {
          key: 'checked',
          label: '',
        },
        {
          key: 'title',
          label: this.$t('Task'),
        }
      ],
    }
  },
  watch: {
    searchByName() {
      this.searchTitle()
    },
  },
  mounted() {
    this.getUserStory()
    this.getTasks(1)
  },
  methods: {
    ...modalManager,
    modal(value, task) {
      this.open({ name: value, object: task })
    },

    getUserStory() {
      Axios()
        .get('user-stories/' + this.$route.params.userStorySlug + 
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug)
        .then((response) => {
          this.data = response.data.data
        })
    },
    getTasks(page) {
      this.gotoScrollTop()
      
      if ( !this.btnLoading ){
        this.loading = true
      }

      Axios()
        .get(
          'tasks/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&title=' + 
            this.filterTitle +
            '&page=' +
            page
        )
        .then((response) => {
          this.tasks = response.data.data
          this.filtered = this.tasks
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page

          for (let i = 0; i < this.tasks.length; i++) {
            this.items[this.tasks[i].uuid] = this.tasks[i].user_story.slug !== null
          }

          this.assignedTasks = this.filtered.filter((data) => data.user_story.title)
          this.loading = false
          this.btnLoading = false
        })
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
    },

    updatePriority(priority) {
      if (priority){
        this.data.priority.color = priority.color
        let params = {
          user_story_priority_id: priority.code,
        }

        this.update(params)
      }
    },
    assignTask(item, id) {
      this.assignLoading = true
      Axios()
        .put(
          'tasks/' +
            item.uuid +
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug,
          {
            user_story_id: id // this.data.id,
          }
        )
        .then((response) => {
          this.filtered = this.tasks
          this.assignedTasks = this.filtered.filter((data) => data.user_story.title)
          this.assignLoading = false
          this.loading = false
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

    sendAssign(item, index) {
      this.currentItem = index
      this.assignLoading = true

      if ( !this.items[item.uuid] ){
        this.assignTask(item, this.data.id)
        this.items[item.uuid] = true
        return true
      }

      this.assignTask(item, null)
      this.items[item.uuid] = false
      return false
    },

    search(){
      this.btnLoading = true
      this.getTasks()
    }
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
     <TitleLoading
        :title="$t('User Stories')"
        :subtitle="$t('User stories are short and simple descriptions of capabilities')">
      </TitleLoading>
    </template>

    <div slot="content" class="user-story pt-10px">
      
      <b-container>
        <b-row>
          <b-col>
            <b-card>
              <template v-slot:header>
                <div class="d-flex justify-content-between align-items-center">
                  <span v-text="$t('User Story')"></span>
                  <router-link
                  :to="{
                    name: 'projects.user-stories.show',
                    params: { projectSlug: data.project.slug, sprintSlug: data.slug } }" class="small" >
                  {{ $t('Go back to User Story') }}
                </router-link>
                </div>
              </template>
              <span class="vlabeledit-label" v-text="data.title" />
            </b-card>
          </b-col>
        </b-row>
        <b-row>
          <b-col class="d-flex justify-content-between align-items-center p-3">
            <TitleLoading
              :title="$t('Assign tasks to User Story')"
              :loading="loading"></TitleLoading>
            <div>
            <b-input-group>
              <b-input-group-append>
                <b-form-input 
                v-model="filterTitle" 
                :placeholder="$t('Search')"
                type="search" size="sm"></b-form-input>
                <ButtonLoading
                :loading="btnLoading"
                type="btn-sm"
                icon="search"
                @action="search"
                ></ButtonLoading>
              </b-input-group-append>
            </b-input-group>
            </div>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-table class="table-assign-task" hover :items="filtered" :fields="fields" >
              <template v-slot:cell(checked)="data" >
                <b-form-checkbox
                v-model="items[data.item.uuid]"
                :disabled="assignLoading"
                value="true" @change="sendAssign(data.item, data.index)"></b-form-checkbox>
                <b-spinner
                v-show="assignLoading && currentItem === data.index"
                :label="$t('Loading')"
                small></b-spinner>
              </template>
              <template v-slot:cell(title)="data" >
                <ListTasks :items="[data.item]" :modal-flag="false"></ListTasks>
              </template>
            </b-table>
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
