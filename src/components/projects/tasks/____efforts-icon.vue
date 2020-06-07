<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import TitleLoading from '@components/utils/title-loading'
import { taskManager, modalManager } from '@state/helpers'


export default {
  components: { TitleLoading, ButtonLoading },
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
      btnLoading: false,
      efforts: [],
      currentPage: 0,
      totalPages: 1,
      searchTerm: '',
      searchLoading: false,
    }
  },
  methods: {
    ...taskManager,
    ...modalManager,

    modal(value) {
      this.currentPage = 0
      this.open({ name: value, object: [] })
    },

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loading = true
        this.getTaskEfforts()
      }
    },

    getTaskEfforts() {
      Axios()
        .get(
          'project-templates/effort/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&search=' + 
            this.searchTerm +
            '&page=' +
            this.currentPage
        )
        .then((response) => {

          this.efforts = response.data.data
          this.searchLoading = false
          this.loading = false
        })
    },

    changeTaskEffort(effort) {
      
      this.actionTask({ name: 'effort.change', uuid: this.task.uuid, object: effort})

      Axios()
      .put(
        'tasks/' +
          this.task.uuid +
          '/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug,
        {
          config_issue_effort_id: effort.id,
        }).then((response) => {
          this.task.effort = effort
        })

    },
    search() {
      this.searchLoading = true
      this.types = []
      this.currentPage = 1
      this.getTaskEfforts()
    },
  },
}
</script>

<template>
<div class="mb-8-px">

  <b-dropdown v-if="authorize('tasks', 'create')" ref="dropdown" right class="dropdown-400px styled-dropdown" @shown="loadMore">
    <template v-slot:button-content >
      <span class="icon-size"><font-awesome-icon :icon="['far', 'dumbbell']" style="font-size:14px"/></span>
      <span>{{ $t('Task Effort') }}</span>
    </template>
    <b-dropdown-header>
      <div class="d-flex align-items-center justify-content-between">
        <TitleLoading :loading="loading" :title="$t('Task Effort')"></TitleLoading>
        
        <div class="dropdown-header-icons">
          <div>
            
            <font-awesome-icon :icon="['far', 'plus-square']" />
            <span>{{ $t('Create Task Effort') }}</span>
           
          </div>
        </div>
      </div>
    </b-dropdown-header>
    <b-dropdown-form class="pb-10-px">
  
      <div class="input-group-search">
        <b-form-input v-model="searchTerm" 
          autocomplete="off" 
          :placeholder="$t('Search effort')"
          @keydown.enter.prevent="search">
        </b-form-input>

        <ButtonLoading
          type="btn-sm"
          mode="button"
          :loading="searchLoading"
          :fa-icon="'far'"
          :icon="'search'"
          @action="search">
        </ButtonLoading>
      </div>
      
    </b-dropdown-form>
    <div class="scroll-activate-h200">
      <b-dropdown-item 
        v-for="effort in efforts"
        :key="effort.id"
        
        @click="changeTaskEffort(effort)">
        <div class="d-flex justify-content-start">
          {{ effort.title }} - {{ effort.effort }} {{ $t('points') }}
        </div>
      </b-dropdown-item>
    </div>
  </b-dropdown>
</div>
</template>
