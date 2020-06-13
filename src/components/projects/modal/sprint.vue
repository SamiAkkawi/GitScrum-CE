<script>
import Axios from '@utils/axios'
import DatePicker from 'vue2-datepicker'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'

export default {
  components: { DatePicker, ButtonLoading },
  data() {
    return {
      loading: false,
      projectLoading: false,
      sprintName: '',
      sprintGoal: '',
      sprintDates: new Date(),
      sprintStatus: null,
      sprintStatuses: [],
      project: [],
      projects: [],
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      errors: [],
    }
  },
  computed: {
    ...modalManager,
  },
  watch: {
    statusModal(object) {
      if (object.item.name === 'sprintCreate') {
        object.item.name = ''
        this.errors = []
        this.getProjects()
        this.getSprintStatuses()

        this.$refs['modal'].show()
      }
    },
  },
  methods: {
    hideModal() {
      this.closeModal(this.$refs['modal'])
    },
    getProjects() {
      Axios()
        .get('projects/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.projects = response.data.data
          if ( this.$route.params.projectSlug ){
            this.project = this.$route.params.projectSlug
          } else {
            this.project = this.projects[0].slug
          }
          this.projectLoading = false
        })
    },

    getSprintStatuses() {
      Axios()
        .get('sprints/statuses?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.sprintStatuses = response.data.data
          this.sprintStatus = response.data.data[0].slug
        })
    },

    create(event) {
      
      this.loading = true

      let dateStart = new Date(this.sprintDates[0])
      let dateStartFormated = dateStart.getFullYear() + '-' + (dateStart.getMonth() + 1) + '-' + dateStart.getDate()

      let dateFinish = new Date(this.sprintDates[1])
      let dateFinishFormated = dateFinish.getFullYear() + '-' + (dateFinish.getMonth() + 1) + '-' + dateFinish.getDate()

      Axios()
        .post('sprints/?company_slug=' + 
          this.currentCompany.slug + 
          '&project_slug=' + this.project, 
        {
          title: this.sprintName,
          date_start: dateStartFormated,
          date_finish: dateFinishFormated,
          description: this.sprintGoal,
          sprint_status_id: this.sprintStatus,
        })
        .then((response) => {
          let data = response.data
          this.loading = false
          this.$router.push({
            name: 'projects.sprints.show',
            params: {
              companySlug: this.currentCompany.slug,
              projectSlug: data.project.slug,
              sprintSlug: data.slug,
            },
          })
        })
    },
  },
}
</script>

<template>
  <b-modal id="modal-black" ref="modal" hide-header hide-footer size="xx">
    <b-container>
      <b-row class="mb-3">
        <b-col>
          <h3 class="mb-0">{{ $t('Create a Sprint') }}</h3>
          <span>{{ currentCompany.name }}</span>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Project Name')">
            <b-form-select 
            v-model="project" 
            :options="projects" 
            value-field="slug"
            text-field="name"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Project Status')">
            <b-form-select 
            v-model="sprintStatus" 
            :disabled="projectLoading"
            :options="sprintStatuses" 
            value-field="slug"
            text-field="title"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Sprint Name')">
            <b-form-input 
            v-model="sprintName" 
            :disabled="projectLoading"
            type="text" 
            maxlength="60" 
            trim></b-form-input>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Project Name')">
            <DatePicker
            v-model="sprintDates"
            :disabled="projectLoading"
            range
            lang="en"
            format="YYYY-MM-DD"
            confirm
            style="width:100%"></DatePicker>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Sprint Goals')">
            <b-form-textarea 
            v-model="sprintGoal" 
            :disabled="projectLoading"
            rows="1" 
            max-rows="4"></b-form-textarea>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col align-self="center">
          <b-link 
            v-show="!loading" 
            @click="hideModal" 
            v-text="$t('Cancel')" />
        </b-col>
        <b-col class="text-right">
          <ButtonLoading
            :loading="loading"
            :title="$t('Create Sprint')"
            :title-loading="$t('Creating')"
            type="btn-md"
            mode="button"
            @action="create"></ButtonLoading>
        </b-col>
      </b-row>
    </b-container>
  </b-modal>
</template>
