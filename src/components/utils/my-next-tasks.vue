<script>
import Axios from '@utils/axios'
import Alert from '@components/utils/alert'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import ListTasks from '@components/projects/tasks/list-tasks'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'

export default {
  page: {
    title: 'Dashboard',
    meta: [{ name: 'description', content: 'ProjectsList' }],
  },
  components: {
    Alert,
    vSelect,
    TitleLoading,
    ListTasks,
    ButtonLoading
  },
  data() {
    return {
      loadingTask: false,
      btnLoading: false,
      projects: [],
      projectSelected: null,
      tasks: [],
      projectsTasks: [],
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
      currentUser: JSON.parse(window.localStorage.getItem('CURRENT_USER')),
      calendarActivities: [],
      showCompanyActivity: true,
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      searchTitle : ''
    }
  },
  created(){
    this.getProjects()
    this.getTasks(this.currentPage)
  },
  methods: {
    getTasks(page) {
      
      if (!this.loadingTask){

        this.loadingTask = true
        
        let projectSlug = null;

        if ( this.projectSelected ){
          projectSlug = this.projectSelected.value
        }

        Axios()
        .get('tasks?assignee=me&workflow=open&company_slug=' + this.currentCompany.slug + 
          '&project_slug=' + projectSlug + 
          '&title=' + this.searchTitle + 
          '&page=' + page)
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getTasks(1)
            return
          }

          if (!this.projectSelected){
            this.projectSelected = this.projects[0]
          }

          this.tasks = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loadingTask = false

        })

      }
    },
    getProjects() {
      
      Axios()
        .get('projects?company_slug=' + this.currentCompany.slug)
        .then((response) => {

          let data = response.data.data
          
          this.projects.push({    
            value: null,
            text: this.$t('All Projects')
          })
          data.forEach((item, key) => {
            
            this.projects.push({
              value: item.slug,
              text: item.name
            })

          })

          this.projectSelected = this.projects[0]

        })
        .catch((e) => {
          console.error(e)
        })
    },
  },
}
</script>

<template>
    <div class="my-next-tasks">
      
      <div class="my-next-tasks-header shadow-sm">
        
        <div class="d-flex justify-content-between align-items-start">
          <TitleLoading :title="$t('My Next Tasks')" :loading="loadingTask"></TitleLoading>
          <div v-b-tooltip.hover :title="$t('Refresh tasks')" class="mr-2 cursor-pointer" @click="getTasks">
            <font-awesome-icon :icon="['far', 'sync-alt']" style="font-size:14px;color: #909CB8;position:relative; top:-2px;" />
          </div>
        </div>

        <b-input-group>
          <v-select
            v-model="projectSelected"
            :options="projects"
            label="text"
            class="mr-1"
            @input="getTasks"
          ></v-select>
          <b-input-group-append>
            <b-form-input 
            v-model="searchTitle" 
            :placeholder="$t('Search tasks by title')"
            type="search" size="sm"></b-form-input>
            <ButtonLoading
              :loading="btnLoading"
              type="btn-sm"
              icon="search"
              @action="getTasks"
            ></ButtonLoading>
          </b-input-group-append>
        </b-input-group>

        

      </div>

      <div class="box-list-task">
        <ListTasks
          id="task-assignees"
          class="mt-20-px"
          :items="tasks"
          :search="true"
          title=""
          :flag="true"
          :display-assignees="false"
        ></ListTasks>

        <div v-if="!tasks.length && !loadingTask">
          <Alert :message="$t('You have no tasks to do on this project')" :status="true"></Alert>
        </div>

        <div v-if="totalPages > 1" class="d-flex justify-content-center mt-4">
          <b-pagination
            v-model="currentPage"
            hide-goto-end-buttons
            class="paginator"
            :total-rows="totalRows"
            :per-page="perPage"
            @change="getTasks"
          >
            <template slot="prev-text">
              <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
            </template>
            <template slot="next-text">
              <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
            </template>
          </b-pagination>
        </div>
      </div>

    </div>
</template>
