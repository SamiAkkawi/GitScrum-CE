<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import Pagination from '@components/utils/pagination'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'User Story',
    meta: [{ name: '', content: '' }],
  },
  components: { Layout, ListUsers, TitleLoading, ButtonLoading, Pagination },
  data() {
    return {
      loading: true,
      searchLoading: false,
      userStories: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      seachByProjectName: '',
      searchTerm: '',

      fields: [
        {
          key: 'priority',
          label: this.$t('Priority'),
        },
        {
          key: 'title',
          label: this.$t('User Story'),
        },
      ],
    }
  },
  mounted() {
    this.getUserStories(this.currentPage)
  },
  methods: {
    ...modalManager,

    modal(value) {
      this.open({ name: value, object: [] })
    },
    getUserStories(page) {
      this.gotoScrollTop()
      this.loading = true
      Axios()
        .get(
          'user-stories/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&search=' + 
            this.searchTerm +
            '&page=' +
            page
        )
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getUserStories(1)
            return
          }
          this.userStories = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loading = false
          this.searchLoading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },

    search() {
      this.searchLoading = true
      this.userStories = []
      this.currentPage = 1
      this.getUserStories(1)
      this.loading = false
    },
  },
}
</script>

<template>
  <Layout>
    
    <template slot="header-left">
     <TitleLoading
        :title="$tc('Explore User Stories', totalRows)"
        :title-alternative="$t('User Stories')"
        :subtitle="$t('User stories are short and simple descriptions of capabilities')"
        :loading="loading"
      ></TitleLoading>
    </template>

    <div slot="content" class="user-story pt-10px">
      
      <div class="container">
        
        <div class="d-flex justify-content-between mb-3">
          
          <div class="d-flex justify-content-end">
            <div class="form-group m-0">
              <div class="input-group">
                <input
                  v-model="searchTerm"
                  maxlength="45"
                  type="text"
                  class="form-control"
                  autocomplete="off"
                  :placeholder="$t('Search user story')"
                  style="height: 32px"
                  @keydown.enter.prevent="search" />
                <div class="input-group-append">
                  <ButtonLoading
                    type="btn-sm"
                    mode="button"
                    :loading="searchLoading"
                    :fa-icon="'far'"
                    :icon="'search'"
                    @action="search"></ButtonLoading>
                </div>
              </div>
            </div>
            <div v-if="authorize('userStories', 'create')" class="ml-20-px">
              <button slot="button" type="button" class="btn btn-primary btn-sm" @click="modal('userstoryCreate')">
                {{ $t('Create an User Story') }}
              </button>
            </div>
          </div>
        </div>

        <b-table class="table-discussions"  hover :items="userStories" :fields="fields" >
          <template v-slot:cell(priority)="data" >
            <span
              v-if="data.item.priority"
              class="badge badge-primary"
              :style="
                'color: ' + invertColor(data.item.priority.color, true) + 
                ';background:' + data.item.priority.color">
              {{ data.item.priority.title }}
            </span>
          </template>
          <template v-slot:cell(title)="data" >
            <div>
              <router-link
                :to="{
                  name: 'projects.user-stories.show',
                  params: {
                    companySlug: data.item.company.slug,
                    projectSlug: data.item.project.slug,
                    userStorySlug: data.item.slug,
                  },
                }" class="txt-primary-title txt-link">
                {{ data.item.code }} - {{ data.item.title }}
              </router-link>
              <p class="mb-0">
                <span>{{ $t('Tasks') }}: {{ data.item.stats.tasks }}</span>
                -
                <span>
                  {{ $tc('Story Point', 2) }}:
                  {{ data.item.stats.story_points }}
                </span>
                -
                <span>
                  {{ $t('Hours Worked') }}:
                  {{ data.item.stats.worked_hours }}
                </span>
              </p>
            </div>
            <div class="mt-1 d-flex align-items-center justify-content-start">
              <ListUsers :users="data.item.users" :link="true" :limit="30" :wrap="true" :size="22"></ListUsers>
            </div>
          </template>
        </b-table>

        <Pagination 
          :total-pages="totalPages" 
          :page="currentPage" 
          :total-rows="totalRows" 
          :per-page="perPage" 
          @change="getUserStories"></Pagination>
          
      </div>
    </div>
  </Layout>
</template>
