<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import TitleLoading from '@components/utils/title-loading'

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
    // refresh: {
    //   type: Array,
    //   required: true,
    // },
    activities: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      loading: true,
      btnLoading: false,
      selectSprintLoading: false,
      sprints: [],

      currentPage: 0,
      totalPages: 1,
      perPage: 15,
      loadingMore: true,
      searchTerm: '',
      termSearched: false,
      searchLoading: false,
    }
  },
  methods: {

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loadingMore = true
        this.getSprints()
      } else {
        this.loadingMore = false
      }
    },

    getSprints() {
      Axios()
        .get(
          'sprints/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&search=' + 
            this.searchTerm +
            '&page=' +
            this.currentPage
        )
        .then((response) => {
          this.currentPage = response.data.current_page
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          if (this.sprints.length && this.currentPage > this.total_pages) {
            this.currentPage = 1
            this.getSprints()
          } else {
            for (let i = 0; i < response.data.data.length; i++) {
              let sprint = response.data.data[i]
              this.sprints.push(sprint)
            }
          }
          this.searchLoading = false
          this.loading = false
        })
    },

    changeTaskSprint(sprint) {
      if( !this.selectSprintLoading ){
        sprint.selectSprintLoading = true
        Axios()
          .put(
            'tasks/' +
              this.task.uuid +
              '/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug,
            {
              sprint_id: sprint.id,
            }
          )
          .then((response) => {
            this.task.sprint = sprint
            sprint.selectSprintLoading = false
          })
      }
    },

    search() {
      this.searchLoading = true
      this.sprints = []
      this.currentPage = 1
      this.termSearched = true
      this.getSprints()
    },
  },
}
</script>

<template>
<div>
  <b-button 
  v-if="authorize('tasks', 'create')" 
  v-b-toggle.sprints-icon 
  class="btn btn-secondary btn-block"
  :style="(task.type) ? 'color: ' + 
  invertColor(task.type.color, true) + 
  ';background: ' + 
  opacityColor(task.type.color, '0.6') : ''"
  v-text="$t('Sprints')"></b-button>
  <b-collapse id="sprints-icon" @shown="loadMore">
    <b-card>
    <b-spinner 
      v-if="loading" 
      :label="$t('Loading')" 
      tag="div" small class="mt-1 ml-1 mb-1"></b-spinner>
    <div v-if="!loading" >
      <b-input-group class="mb-2">
        <b-input-group-append class="wd-100">
          <b-form-input 
          v-model="searchTerm" 
          maxlength="45"
          :placeholder="$t('Search')"
          type="search" size="sm"></b-form-input>
          <ButtonLoading
            :loading="searchLoading"
            type="btn-sm"
            icon="search"
            @action="search"
          ></ButtonLoading>
        </b-input-group-append>
      </b-input-group>
      
      <div v-if="sprints.length" class="scroll-activate-h200">
        <div v-for="sprint in sprints" 
          v-if="sprint.id !== task.sprint.id" 
          :key="sprint.id" 
          class="mb-2 pb-2 border-bottom">
          <b-link class="small txt-primary" @click="changeTaskSprint(sprint)" v-text="sprint.title"></b-link>
          <div class="d-flex justify-content-between">
            <div class="small text-muted">{{ sprint.timebox }} - {{ sprint.duration }}</div>
            <b-spinner
              v-show="selectSprintLoading === sprint.id"
              :label="$t('Loading')"
              small
              class="mt-2 mr-2" ></b-spinner>
          </div>
        </div>
        <b-link 
        v-if="loadingMore" 
        class="txt-primary small font-weight-bold" 
        @click="loadMore">{{ $t('Load more') }}</b-link>
      </div>
    </div>
    </b-card>
  </b-collapse>
</div>
</template>
