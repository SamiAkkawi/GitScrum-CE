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
    activities: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      loading: true,
      btnLoading: false,
      selectUserStoryLoading: false,
      userStories: [],

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
        this.getUserStories()
      } else {
        this.loadingMore = false
      }
    },

    getUserStories() {
      Axios()
      .get(
        'user-stories/?company_slug=' + 
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
        if ( !this.userStories && this.currentPage > this.total_pages) {
          this.currentPage = 1
          this.getUserStories()
        } else {
          for (let i = 0; i < response.data.data.length; i++) {
            let userStory = response.data.data[i]
            this.userStories.push(userStory)
          }
        }

        this.searchLoading = false
        this.loading = false
      })
    },

    changeTaskUserStory(userStory) {
      if( !this.selectUserStoryLoading ){
        this.selectUserStoryLoading = userStory.id
        Axios()
        .put(
          'tasks/' +
          this.task.uuid +
          '/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug,
          {
            user_story_id: userStory.id,
          }
        )
        .then((response) => {
          this.task.user_story = userStory
          this.selectUserStoryLoading = false
        })
      }
    },

    search() {
      this.searchLoading = true
      this.userStories = []
      this.currentPage = 0
      this.termSearched = true
      this.getUserStories()
    },
  },
}
</script>

<template>
  <div>
    <b-button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.user-story-icon
    class="btn btn-secondary btn-block"
    :style="(task.type) ? 'color: ' + 
    invertColor(task.type.color, true) + 
    ';background: ' + 
    opacityColor(task.type.color, '0.6') : ''"
    v-text="$t('User Stories')"></b-button>
    <b-collapse id="user-story-icon" @shown="loadMore">
      <b-card>
        <b-spinner 
          v-if="loading" 
          :label="$t('Loading')" 
          tag="div" small class="mt-1 ml-1 mb-1"></b-spinner>
        <div v-if="!loading">
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
          <div v-if="userStories.length" class="scroll-activate-h200">
            <div v-for="userStory in userStories" 
              v-if="userStory.id !== task.user_story.id" 
              :key="userStory.id" 
              class="mb-2 pb-2 border-bottom">
              <b-link class="small txt-primary" @click="changeTaskUserStory(userStory)" v-text="userStory.title"></b-link>
              <div class="d-flex justify-content-between">
                <span v-if="userStory.priority" class="badge mt-1" 
                :style="'color: ' + 
                invertColor(userStory.priority.color, true) + 
                ';background:' + 
                userStory.priority.color"
                v-text="userStory.priority.title" />
                <b-spinner
                  v-show="selectUserStoryLoading === userStory.id"
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
