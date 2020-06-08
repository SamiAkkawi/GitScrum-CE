<script>
import Axios from '@utils/axios'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'
import ButtonLoading from '@components/utils/button-loading'
import TitleLoading from '@components/utils/title-loading'
import { taskManager } from '@state/helpers'

export default {
  components: { vSelect, TitleLoading, ButtonLoading },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
    checklistId: {
      type: Number,
      required: false,
      default: 0,
    },
  },
  data() {
    return {
      loading: false,
      btnLoading: false,
      btnTemplateLoading: false,
      title: '',
      alertMessage: '',
      alertStatus: false,
      boxSelected: 'create',
      templateItem: null,
      templateItems: [],
    }
  },
  methods: {
    ...taskManager,

    addChecklist() {
      this.btnLoading = true
      this.alertStatus = false
      Axios()
        .post('task-checklists/?company_slug=' + this.task.company.slug + '&project_slug=' + this.task.company.slug, {
          parent_id: this.checklistId,
          task_uuid: this.task.uuid,
          title: this.title,
        })
        .then((response) => {
          this.title = ''
          this.btnLoading = false
          this.actionTask({ name: 'checklist.addList', object: response.data.data })
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
        })
    },

    getTemplateTaskChecklists() {
      this.loading = true
      this.alertStatus = false
      Axios()
        .get(
          'project-templates/checklist/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          this.templateItems = response.data.data
          this.templateItem =  response.data.data[0]
          this.loading = false

          if (this.templateItems.length === 0) {
            this.alertMessage = this.$t('Dont have any checklist to apply')
            this.alertStatus = true
          }
        })
        .catch((e) => {
          this.loading = false
        })
    },

    applyTemplate() {
      this.btnTemplateLoading = true

      Axios()
        .post(
          'task-checklists/' +
            this.templateItem +
            '/apply/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug,
          {
            task_uuid: this.task.uuid,
          }
        )
        .then((response) => {
          this.btnTemplateLoading = false
          this.actionTask({ name: 'checklist.reload' })
        })
        .catch((e) => {
          this.btnTemplateLoading = false
        })
    },

    openCreate() {
      this.boxSelected = 'create'
    },

    openTemplate() {
      this.boxSelected = 'template'
      this.getTemplateTaskChecklists()
    },
  },
}
</script>

<template>
  <div>
    <b-button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.checklist-icon 
    class="btn btn-secondary btn-block"
    :style="(task.type) ? 'color: ' + 
    invertColor(task.type.color, true) + 
    ';background: ' + 
    opacityColor(task.type.color, '0.6') : ''"
    v-text="$t('Checklist')"></b-button>
    <b-collapse id="checklist-icon">
      <b-card>
        <b-link v-b-toggle.checklist-template class="badge badge-light mb-2">
          <font-awesome-icon :icon="['far', 'boxes']" class="mr-1" />
          {{$t('Use Checklist Template')}}
        </b-link>
        <b-collapse id="checklist-template" class="mb-2" @shown="getTemplateTaskChecklists">
          <b-form-select 
          v-model="templateItem" 
          :options="templateItems"
          value-field="id"
          text-field="title" 
          size="sm"></b-form-select>
          <div class="d-flex justify-content-end mt-1">
            <ButtonLoading
              type="btn-md"
              mode="button"
              :loading="btnTemplateLoading"
              :title="$t('Add Template Checklist')"
              :title-loading="$t('Adding')"
              @action="applyTemplate"></ButtonLoading>
          </div>
        </b-collapse>
        <b-input-group>
          <b-input-group-append class="wd-100">
            <b-form-input 
            v-model="title" 
            size="sm" 
            :placeholder="$t('Write a title for checklist')"></b-form-input>
            <ButtonLoading
            :loading="btnLoading"
            type="btn-sm"
            icon="plus"
            @action="addChecklist"></ButtonLoading>
          </b-input-group-append>
        </b-input-group>
      </b-card>
    </b-collapse>
  </div>
</template>
