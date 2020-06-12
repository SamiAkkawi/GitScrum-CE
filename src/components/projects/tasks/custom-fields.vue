<script>
import Axios from '@utils/axios'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'

export default {
  components: { vSelect },
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
      fields: [],
      customField: [],
      selectValues: [],
      selectableOptions: [],
      loading: true,
      visible: true,
    }
  },
  created() {
    this.getCustomFields()
  },
  methods: {
    
    getCustomFields() {
      Axios()
      .get(
        'task-fields/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&task_uuid=' +
          this.task.uuid 
      )
      .then((response) => {
        this.fields = response.data.data
        this.handleValues()
        this.loading = false
      })
    },

    handleValues() {
      for (let i = 0; i < this.fields.length; i++) {
        if (this.fields[i].meta) {
          let selectableOptions = []
          if ( this.fields[i].type === 'select' ){
            selectableOptions = this.fields[i].meta.split(',');
            this.$set(this.fields[i], 'selectableOptions', selectableOptions)
          }
        }
      }
    },

    update(param) {
      Axios().post(
        'task-fields/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&task_uuid=' +
          this.task.uuid ,
        param
      )
    },
    updateField(field) {
      this.update({
        field_id: field.id,
        value: field.value,
      })
    },
    updateFieldCheckbox(field) {
      this.update({
        field_id: field.id,
        value: !field.value,
      })
    },
    updateFieldSelected(field) {
      this.update({
        field_id: field.id,
        value: field.value ? field.value : '',
      })
    },
  },
}
</script>

<template>
  <b-row v-if="fields[0]" class="mb-3">
    <b-col cols="1" class="task-left-icon">
      <font-awesome-icon :icon="['far', 'file-alt']" />
    </b-col>
    <b-col class="task-left-content">
      <h5 class="mb-3">{{ $t('Custom Fields') }}</h5>
      <div v-for="field in fields" :key="field.id" class="d-block">
        <b-form-group v-if="field.type === 'text'" :label="field.name" class="mb-1">
          <b-form-input
          v-model="field.value"
          :disabled="!authorize('tasks', 'update')"
          @change="updateField(field)"></b-form-input>
        </b-form-group>
        <b-form-group v-if="field.type === 'textarea'" :label="field.name" class="mb-1">
          <b-form-textarea
          v-model="field.value"
          rows="3"
          max-rows="6"
          :disabled="!authorize('tasks', 'update')"
          @change="updateField(field)"></b-form-textarea>
        </b-form-group>
        <b-form-checkbox v-if="field.type === 'checkbox'"
          v-model="field.value"
          value="true" 
          class="mb-1" 
          :disabled="!authorize('tasks', 'update')"
          @change="updateFieldCheckbox(field)">
          <label v-text="field.name"></label>
        </b-form-checkbox>
        <b-form-group v-if="field.type === 'select'" :label="field.name" class="mb-1">
          <b-form-select
          v-model="field.value"
          :options="field.selectableOptions"
          value-field="id"
          text-field="label"
          :disabled="!authorize('tasks', 'update')"
          @input="updateFieldSelected(field)"></b-form-select>
        </b-form-group>
      </div>
    </b-col>
  </b-row>
</template>
