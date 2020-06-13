<script>
import Axios from '@utils/axios'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: {
    ButtonLoading,
    Swatches,
  },
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
      statusAssignLoading: false,
      loading: false,
      btnLoading: false,
      btnAddLoading: false,
      boxSelected: 'list',
      labels: [],
      label: this.getLabelDefault(),
      newLabel:  this.getLabelDefault(),
      labelProps: { blank: true, width: 15, height: 15, class: 'm1' },
      alertMessage: '',
      alertStatus: false,
    }
  },
  methods: {
    getLabelDefault() {
      return {
        title: '',
        color: '#000000',
      }
    },

    getAllLabels() {
      this.loading = true
      this.alertStatus = false
      Axios()
        .get(
          'task-labels/not-added' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
        )
        .then((response) => {
          this.labels = response.data.data.filter((item) => !this.task.labels.some((label) => label.id === item.id))
          
          if (!this.labels.length){
            this.openCreate()
          }
          this.loading = false
        })
    },

    addLabel() {
      this.btnAddLoading = true
      Axios()
        .post(
          'task-labels/' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid,
          {
            color: this.newLabel.color,
            title: this.newLabel.title,
          }
        )
        .then((response) => {
          this.task.labels.push(response.data.data)
          this.getAllLabels()
          this.newLabel = this.getLabelDefault()
          this.boxSelected = 'list'
          this.btnAddLoading = false
        })
    },

    editLabel(label) {
      this.btnLoading = true
      
      Axios()
        .put(
          'task-labels/' +
            label.id +
            '/' +
            '?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid,
          {
            color: label.color,
            title: label.title,
          }
        )
        .then((response) => {
          this.getAllLabels()
          this.labels = this.getLabelDefault()
          this.boxSelected = 'list'
          this.btnLoading = false
        })
    },

    assignLabel(label) {
      if (!this.statusAssignLoading) {
        this.statusAssignLoading = true
        Axios()
          .post(
            'task-labels/' +
              '?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug +
              '&task_uuid=' +
              this.task.uuid,
            {
              color: label.color,
              title: label.title,
            }
          )
          .then((response) => {
            this.task.labels.push(response.data.data)
            this.getAllLabels()
            this.labels = this.getLabelDefault()
            this.statusAssignLoading = false
          })
      }
    },

    removeLabel(label) {

      Axios()
      .delete(
        'task-labels/' +
          label.id +
          '/detach/' +
          '?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&task_uuid=' +
          this.task.uuid
      )
      .then((response) => {
          this.getAllLabels()
          this.boxSelected = 'list'
          this.labels = this.getLabelDefault()
          this.task.labels.splice(this.task.labels.indexOf(label), 1)
      })

    },
    deleteLabel(label) {
      this.boxSelected = ''

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), 
      this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
              
          Axios()
          .delete(
            'task-labels/' +
              label.id +
              '/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug +
              '&task_uuid=' +
              this.task.uuid
          )
          .then((response) => {
            this.getAllLabels()
              this.labels = this.getLabelDefault()
              this.task.labels.splice(this.task.labels.indexOf(label), 1)
          })
        
        }
      })

    },
    openEdit(label) {
      this.boxSelected = 'edit'
      this.label = label
    },
    openCreate() {
      if (this.boxSelected === 'create' ){
        this.boxSelected = ''
        return
      }

      this.boxSelected = 'create'
      this.label = this.getLabelDefault()
    },
  },
}
</script>

<template>
<div class="task-labels">
  <b-button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.label-icon 
    class="btn btn-secondary btn-block"
    :style="(task.type) ? 'color: ' + 
    invertColor(task.type.color, true) + 
    ';background: ' + 
    opacityColor(task.type.color, '0.6') : ''"
    v-text="$t('Labels')"></b-button>
  <b-collapse id="label-icon" @shown="getAllLabels">
    <b-card>
      
      <div class="d-flex justify-content-between mb-2">
        <b-link v-b-toggle.label-create>
          <span class="badge badge-light">{{$t('Create a Label')}}</span>
        </b-link>
        <b-spinner 
          v-if="loading" 
          :label="$t('Loading')" 
          tag="div" small class="mt-1 ml-1 mb-1"></b-spinner>
      </div>

      <b-collapse id="label-create">
        <b-card class="mt-2 mb-2">
          <b-input-group>
            <b-input-group-append class="wd-100">
              <Swatches
              v-model="newLabel.color"
              colors="text-advanced"
              popover-to="left"
              :trigger-style="{
                width: '32px',
                height: '31px',
                borderRadius: '0' }"
              class="swatches-input"></Swatches>
              <b-form-input 
              v-model="newLabel.title" 
              size="sm" 
              :placeholder="$t('Label Name')"></b-form-input>
              <ButtonLoading
              :loading="btnAddLoading"
              type="btn-sm"
              icon="plus"
              @action="addLabel"
              ></ButtonLoading>
            </b-input-group-append>
          </b-input-group>
        </b-card>
      </b-collapse>
      
      <b-card v-show="boxSelected === 'edit'" class="mt-2 mb-2">
        <b-input-group>
          <b-input-group-append class="wd-100">
            <Swatches
            v-model="label.color"
            colors="text-advanced"
            popover-to="left"
            :trigger-style="{
              width: '32px',
              height: '31px',
              borderRadius: '0' }" class="swatches-input"></Swatches>
            <b-form-input 
            v-model="label.title" 
            size="sm" 
            :placeholder="$t('Label Name')"></b-form-input>
            <ButtonLoading
            :loading="btnLoading"
            type="btn-sm"
            icon="check"
            @action="editLabel(label)"
            ></ButtonLoading>
          </b-input-group-append>
        </b-input-group>
        
        <div class="project-label-form-actions ">
          <b-link @click="boxSelected = ''">
            {{ $t('Close') }}
          </b-link>
          <b-link @click="deleteLabel(label)">
            {{ $t('Delete') }}
          </b-link>
        </div>
      </b-card>

      
      <div class="styled-dropdown">
        <div class="dropdown-fake label-line">
          
          <p class="font-weight-bold mb-0">{{ $t('Click to add the label') }}</p>

          <div v-for="labelItem in labels" :key="labelItem.id" class="dropdown-item">
            <div class="label-name d-flex align-items-center justify-content-start" @click="assignLabel(labelItem)">
              <span class="square" :style="'background:' + labelItem.color"></span>
              <span class="fw-600 ml-5-px">{{ labelItem.title }}</span>
            </div>
            <div class="action-edit" @click="openEdit(labelItem)">
              <font-awesome-icon :icon="['far', 'pencil']" style="font-size:12px" />
            </div>
          </div>

          <hr>
          <p v-if="task.labels.length" class="font-weight-bold mb-0">{{ $t('Click to remove the label') }}</p>
          <div v-for="taskLabel in task.labels" v-if="task.labels.length" :key="taskLabel.id" class="dropdown-item">
            <div class="label-name d-flex align-items-center justify-content-start" @click="removeLabel(taskLabel)">
              <span class="square" :style="'background:' + taskLabel.color"></span>
              <span class="fw-600 ml-5-px">{{ taskLabel.title }}</span>
            </div>
            <div class="action-edit" @click="openEdit(taskLabel)">
              <font-awesome-icon :icon="['far', 'pencil']" style="font-size:12px" />
            </div>
          </div>
        </div>

      </div>
    </b-card>
  </b-collapse>
</div>

</template>
