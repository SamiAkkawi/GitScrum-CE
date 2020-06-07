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
      loading: false,
      btnLoading: false,
      sprints: [],
      // taskSprint: [],
      alertMessage: '',
      alertStatus: false,
      currentPage: 0,
      totalPages: 1,
      perPage: 15,
      loadingMore: false,
      sprintSelected: [],
      searchTerm: '',
      termSearched: false,
      searchLoading: false,
    }
  },
  methods: {
    handleScroll(ev) {
      if (
        ev.target.scrollHeight - 400 <= ev.target.scrollTop &&
        !this.loadingMore &&
        this.currentPage <= this.totalPages
      ) {
        this.loadMore()
      }
    },

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loadingMore = true
        this.getSprints()
      }
    },

    getSprints() {
      this.loading = true
      this.alertStatus = false
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
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.currentPage = response.data.current_page
            this.totalPages = response.data.total_pages
            this.perPage = response.data.per_page
            if (this.sprints.length && this.currentPage > this.total_pages) {
              this.currentPage = 1
              this.getSprints()
            } else {
              for (let i = 0; i < response.data.data.length; i++) {
                let sprint = response.data.data[i]
                this.$set(sprint, 'selected', false)
                this.$set(sprint, 'selectSprintLoading', false)
                this.sprints.push(sprint)
              }
            }
          }
          this.loadingMore = false
          this.searchLoading = false
          this.loading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
          this.loading = false
          this.loadingMore = false
        })
    },

    selectSprint(sprint) {
      if (this.sprintSelected && this.sprintSelected.selected) {
        let sprintIndex = this.sprints.indexOf(this.sprintSelected)
        if ( this.sprints[sprintIndex] ){
          this.sprints[sprintIndex].selected = false
        }
      }
      sprint.selected = true
      this.sprintSelected = sprint
      this.task.sprint = sprint
    },

    changeTaskSprint(sprint) {
      this.alertStatus = false
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
          if (response.data.message) {
            this.alertMessage = 'Error. ' + response.data.message
            this.alertStatus = true
          } else {
            this.selectSprint(sprint)
          }
          sprint.selectSprintLoading = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. ' + error.response.data.message
          this.alertStatus = true
          sprint.selectSprintLoading = false
        })
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
  <button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.sprints-icon 
    class="btn btn-secondary btn-block">
    {{ $t('Sprints') }}
  </button>
  <b-collapse id="sprints-icon" @shown="loadMore">
    <b-card>
      <div>
  
      <div class="input-group-search">
        <b-form-input v-model="searchTerm" 
          autocomplete="off" 
          :placeholder="$t('Search sprint')"
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
      
    </div>
    <div class="scroll-activate-h200" v-on:scroll.passive="handleScroll">
      <b-dropdown-item 
        v-for="sprint in sprints"
        :key="sprint.id"
        @click="changeTaskSprint(sprint)">
        <span class="d-block tx-12-px fw-500 mb-5-px" :class="sprint.selected ? 'txt-3D4F9F' : 'txt-68748F'">
          {{ sprint.title }}
        </span>
        <div class="d-flex justify-content-between">
          <span class="d-block tx-10-px ls-003" :class="sprint.selected ? 'txt-3D4F9F' : 'txt-68748F'">
            {{ sprint.timebox }} - {{ sprint.duration }}
          </span>
          <font-awesome-icon
            v-if="sprint.selected && !sprint.selectSprintLoading"
            :icon="['far', 'check-double']"
            class="txt-464DEE"
            style="font-size: 12px;"
          />
          <b-spinner
            v-show="sprint.selectSprintLoading"
            :label="$t('Loading')"
            small
            class="btn-small-loading"
          ></b-spinner>
        </div>
      </b-dropdown-item>
    </div>
    </b-card>
  </b-collapse>
</div>
</template>
