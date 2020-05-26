<script>
import Axios from '@utils/axios'
import Ads from '@components/utils/ads'
import Alert from '@components/utils/alert'
import TitleLoading from '@components/utils/title-loading'
import ListTasks from '@components/projects/tasks/list-tasks'
import Activities from '@components/utils/activities'
import GlanceYear from '@components/utils/glance-year'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'

export default {
  page: {
    title: 'Dashboard',
    meta: [{ name: 'description', content: 'ProjectsList' }],
  },
  components: {
    Ads,
    Alert,
    vSelect,
    TitleLoading,
    ListTasks,
    Activities,
    GlanceYear,
  },
  data() {
    return {
      loading: true,
      loadingTask: true,
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
    }
  },
  created(){
     this.getProjects()
    this.getTasks(this.currentPage)
   
  },
  methods: {
    getTasks(page) {
      this.gotoScrollTop();
      this.loadingTask = true
      
      let projectSlug = null;

      if ( this.projectSelected ){
        projectSlug = this.projectSelected.value
      }

      Axios()
        .get('tasks?assignee=me&workflow=open&company_slug=' + this.currentCompany.slug + 
          '&project_slug=' + projectSlug + 
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
        .catch((e) => {
          console.error(e)
        })
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
      
      <TitleLoading :title="$t('My Next Tasks')" :loading="loadingTask"></TitleLoading>

      <v-select
        v-model="projectSelected"
        :options="projects"
        label="text"
        @input="getTasks"
      ></v-select>

      <ListTasks
        id="task-assignees"
        class="mt-20-px"
        :items="tasks"
        :search="true"
        title=""
        :flag="true"
      ></ListTasks>

      <div v-if="!tasks.length && !loadingTask">
        <Alert :message="$t('You have no tasks to do on this project')" :status="true"></Alert>
      </div>

      <div v-if="totalPages > 1" class="d-flex justify-content-center mt-4">
        <b-pagination
          hide-goto-end-buttons
          class="paginator"
          v-model="currentPage"
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
</template>
