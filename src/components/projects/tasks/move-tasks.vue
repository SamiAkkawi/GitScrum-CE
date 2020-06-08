<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: { ButtonLoading },
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
      loading: false,
      projectLoading: true,
      workflowLoading: false,
      visible: false,
      
      openMoveTasks: false,
      
      projects: null,
      workflows: [],
      selectedProject: this.project,
      selectedWorkflow: '',
    }
  },
  methods: {
    moveTask() {
      this.loading = true
      Axios()
        .post('tasks/' + this.task.uuid + '/move', {
          company_slug: this.task.company.slug,
          project_slug: this.task.project.slug,
          new_company_slug: this.task.company.slug,
          new_project_slug: this.selectedProject,
          new_workflow_id: this.selectedWorkflow,
        })
        .then((response) => {
          this.$router.push({
            name: 'projects.board.task-details',
            params: {
              companySlug: this.task.company.slug,
              projectSlug: this.selectedProject,
              taskSlug: this.task.uuid,
            },
          })
        })
    },
    getProjects() {
      if ( !this.projects ){
        this.visible = false
        this.projectLoading = true
        Axios()
        .get('projects/?company_slug=' + 
        this.task.company.slug)
        .then((response) => {
          this.projects = response.data.data
          this.selectedProject = this.projects[0].slug
          this.projectLoading = false
          this.getWorkflows()
        })
      }
    },
    getWorkflows() {
      this.visible = false
      this.workflowLoading = true
      this.workflows = []
      Axios()
      .get(
        'projects-workflows/?company_slug=' + 
        this.task.company.slug + 
        '&project_slug=' + 
        this.selectedProject
      )
      .then((response) => {
        this.workflows = response.data.data
        this.selectedWorkflow = this.workflows[0].id
        this.workflowLoading = false
        this.visible = true
      })
    },
  },
}
</script>

<template>
  <div>
    <b-button 
    v-if="authorize('tasks', 'update')" 
    v-b-toggle.move-task 
    class="btn btn-secondary btn-block"
    :style="(task.type) ? 'color: ' + 
    invertColor(task.type.color, true) + 
    ';background: ' + 
    opacityColor(task.type.color, '0.6') : ''"
    v-text="$t('Move to another Project')"></b-button>
    <b-collapse id="move-task" @shown="getProjects">
      <b-card>
        <b-spinner 
          v-if="projectLoading" 
          :label="$t('Loading')" 
          tag="div" small class="mt-1 ml-1 mb-1"></b-spinner>
        <b-form-group
          v-if="!projectLoading"
          :label="$t('Project')" class="mb-1">
          <b-form-select 
          v-model="selectedProject" 
          :options="projects"
          value-field="slug"
          text-field="name"
          size="sm"
          @change="getWorkflows"></b-form-select>
        </b-form-group>
        <b-spinner 
          v-if="workflowLoading" 
          :label="$t('Loading')" 
          tag="div" small class="mt-2 ml-1 mb-1"></b-spinner>  
        <b-form-group
          v-if="visible"
          :label="$t('Workflow Stage')">
          <b-form-select 
          v-model="selectedWorkflow" 
          :options="workflows"
          value-field="id"
          text-field="title"
          size="sm"></b-form-select>
        </b-form-group>
        <div v-if="visible" class="mt-3 mb-2">
          <hr>
          <div class="d-flex justify-content-between">
            <div></div>
            <ButtonLoading
            :title="$t('Move')"
            :title-loading="$t('Moving')"
            :loading="loading"
            type="btn-md"
            @action="moveTask"></ButtonLoading>
          </div>
        </div>
      </b-card>
    </b-collapse>
  </div>
</template>
