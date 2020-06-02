<script>
import Axios from '@utils/axios'
//import TaskWorkflowCreate from '@components/utils/task-workflow-create'
import TaskWorkflowList from '@components/utils/task-workflow-list'

export default {
  components: { TaskWorkflowList },
  props: {
    templateSelected: {
      type: Object,
      required: true,
    },
    component: {
      type: String,
      required:true,
    }
  },
  data() {
    return {
      btnLoading: false,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
    }
  },
  methods: {
    getTitle(type){
      switch (type) {
        case "workflow":
          return this.$t('Workflow')
        case "type":
          return this.$t('Task Types')
        case "effort":
          return this.$t('Task Efforts')
        case "field":
          return this.$t('Task Custom Fields')
        case "checklist":
          return this.$t('Task Checklists')
        case "priority":
          return this.$t('User Story Priorities')
        default:
          return this.$t('Workflow');
      }
    },

    getComponentName(type, sulfix){
      return require('@components/companies/templates/partials/' + type + '-' + sulfix).default
    },

    updateTemplateName() {
      if (this.templateSelected.name.length) {
        Axios()
          .put('templates/workflow/' + 
            this.templateSelected.id + 
            '?company_slug=' + this.currentCompany.slug, 
          {
            name: this.templateSelected.name,
          })
      }
    },

    updateIsPrivate() {
      Axios()
        .put('templates/' + 
          this.component + '/' + 
          this.templateSelected.id + 
          '/is_private/?company_slug=' + 
          this.currentCompany.slug, 
        {
          is_private: this.templateSelected.is_private
        })
        .then((response) => {
          this.$emit('update', true)
        })
    },

    updateIsDefault() {
      Axios()
        .put('templates/' + 
          this.component + '/' + 
          this.templateSelected.id + 
          '/is_default/?company_slug=' + 
          this.currentCompany.slug, 
        {
          is_default: this.templateSelected.is_default
        })
        .then((response) => {
          this.$emit('update', true)
        })
    },

    deleteSelectedTemplate() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete('templates/' + 
            this.component + 
            '/' + 
            this.templateSelected.id + 
            '/?company_slug=' + 
            this.currentCompany.slug)
          .then((_) => {
            this.$emit('delete', this.templateSelected)
            
          })
        }
      })
    },
  },
}
</script>

<template>
  <b-card>
    <b-card-title>{{ getTitle(component) }}</b-card-title>
    <b-card-text>
      <b-row>
        <b-col cols="10">
          <b-form-input 
            v-model="templateSelected.name" 
            :placeholder="$t('Template Name')"
            @change="updateTemplateName"></b-form-input>
        </b-col>
        <b-col cols="2" class="text-right">
          <b-link class="txt-68748F fw-500 lh-15-px tx-10-px tx-uppercase" 
            href="javascript:;" 
            @click="deleteSelectedTemplate">
            {{ $t('Delete') }}
          </b-link>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="12" class="mt-3">
          <b-form-checkbox
            v-model="templateSelected.is_default"
            value="1"
            unchecked-value="0"
            @change="updateIsDefault">
            <strong>{{ $t('Make this Template as standard for all projects') }}</strong> 
            <font-awesome-icon v-b-popover.hover.top="$t('You can choose only one template standard per company. This template will be applied whenever you create a new project.')" 
              :icon="['fal', 'question-circle']" 
              class="ml-2" />
          </b-form-checkbox>
          <b-form-checkbox
            v-model="templateSelected.is_private"
            value="1"
            unchecked-value="0"
            @change="updateIsPrivate">
            {{ $t('Share this Template on GitScrum Marketplace') }}
          </b-form-checkbox>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="12">
          <hr>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="12">
          <component 
          :is="getComponentName(component, 'create')" 
          :current-company="currentCompany"
          :template-selected="templateSelected"></component>
          <hr class="px-1">
          <component 
            :is="getComponentName(component, 'list')" 
            :current-company="currentCompany"
            :template-selected="templateSelected"></component>
        </b-col>
      </b-row>
      
    </b-card-text>
  </b-card>
</template>
