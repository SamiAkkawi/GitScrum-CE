<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { taskManager } from '@state/helpers'

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
      loading: true,
      btnLoading: false,
      subtaskTitle: '',
      alertMessage: '',
      alertStatus: false,
      workflow: '',
      workflows: null
    }
  },
  methods: {
    ...taskManager,

    getWorkflows() {
      if (!this.workflows) {
        this.loading = true
        Axios()
        .get(
          'project-templates/workflow/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
        )
        .then((response) => {
          this.workflows = response.data.data
          this.workflow = this.workflows[0].id
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
      }
    },

    createTask() {
      this.btnLoading = true
      this.alertStatus = false

      if (this.subtaskTitle.length > 3) {
        this.showAddCardLoading = true
        this.showAddCard = false
        Axios()
          .post('/tasks/?company_slug=' + 
          this.task.company.slug + 
          '&project_slug=' + 
          this.task.project.slug, {
            title: this.subtaskTitle,
            parent_id: this.task.uuid,
            config_workflow_id: this.workflow,
          })
          .then((response) => {
            
            this.subtaskTitle = ''
            this.btnLoading = false
            this.actionTask({ name: 'subtask.addList', object: response.data.data })

          })
      } else {
        // alert('You must fill in the task title')
        this.alertMessage = this.$t('Subtask name must to have at least 4 characters')
        this.alertStatus = true
        this.btnLoading = false
      }
    },
  },
}
</script>

<template>
<div>
  <b-button 
  v-if="authorize('tasks', 'create')" 
  v-b-toggle.subtasks-icon 
  class="btn btn-secondary btn-block"
  :style="(task.type) ? 'color: ' + 
  invertColor(task.type.color, true) + 
  ';background: ' + 
  opacityColor(task.type.color, '0.6') : ''"
  v-text="$t('Subtasks')"></b-button>
  <b-collapse id="subtasks-icon" @shown="getWorkflows">
    <b-card>
      <b-spinner 
        v-if="loading" 
        :label="$t('Loading')" 
        tag="div" small class="mt-1 ml-1 mb-1"></b-spinner>
      <b-input-group v-if="!loading" >
        <b-form-select 
          v-model="workflow" 
          :options="workflows"
          value-field="id"
          text-field="title" 
          size="sm"></b-form-select>
        <b-input-group-append class="mt-2 wd-100">
          <b-form-input 
          v-model="subtaskTitle"  
          :placeholder="$t('Write a title for subtask')"
          size="sm"></b-form-input>
          <ButtonLoading
          :loading="btnLoading"
          type="btn-sm"
          icon="plus"
          @action="createTask"></ButtonLoading>
        </b-input-group-append>
      </b-input-group>
    </b-card>
  </b-collapse>
</div>
</template>
