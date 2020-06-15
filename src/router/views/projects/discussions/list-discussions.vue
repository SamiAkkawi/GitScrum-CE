<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import TitleLoading from '@components/utils/title-loading'
import CommentEditor from '@components/utils/comment-editor'
import ButtonLoading from '@components/utils/button-loading'
import Pagination from '@components/utils/pagination'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'Discussions',
    meta: [{ name: '', content: '' }],
  },
  components: { Layout, ListUsers, ButtonLoading, TitleLoading, CommentEditor, Pagination },
  data() {
    return {
      loading: true,
      loadingPriorities: true,
      searchLoading: '',
      comments: [],
      totalRows: 0,
      totalPages: 0,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      btnLoading: false,
      txtComment: '',
      createDiscussionOpen: false,
      filter: null,
      seachByProjectName: '',
      searchTerm: '',
      fields: [
        {
          key: 'comment',
          label: this.$t('Discussions'),
        },
      ],
    }
  },
  mounted() {
    this.getDiscussions(this.currentPage)
  },
  methods: {
    ...modalManager,

    modal(value) {
      this.open({ name: value, object: [] })
    },
    getDiscussions(page) {
      this.gotoScrollTop()
      this.loading = true
      Axios()
        .get(
          'comments/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&commentable_type=projects' +
            '&commentable_id=null' +
            '&search=' + 
            this.searchTerm +
            '&page=' +
            page
        )
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getDiscussions(1)
            return
          }
          this.comments = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loading = false
          this.searchLoading = false
        })
        .catch((error) => {
          console.error(error)
        })
    },

    search() {
      this.searchLoading = true
      this.comments = []
      this.currentPage = 1
      this.getDiscussions(1)
      this.loading = false
    },

    created(){
      this.getDiscussions(1)
    },

    cancel(){
      this.createDiscussionOpen = false
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
     <TitleLoading
        :title="$tc('Explore Discussions', totalRows)"
        :title-alternative="$t('Discussions')"
        :subtitle="$t('Discussion is essential to promote dialogue for project’s goals')"
      >
      </TitleLoading>
    </template>

    <div slot="content" class="project-discussions pt-10px">
       <div class="container">
        <div class="row mb-30-px">
          <div class="col-md-12 pr-0">
            <div class="d-flex justify-content-between mb-3">
              
              <div class="d-flex justify-content-end">
                <div>
                  <div class="form-group m-0">
                    <div class="input-group">
                      <input
                        v-model="searchTerm"
                        maxlength="45"
                        type="text"
                        class="form-control"
                        autocomplete="off"
                        :placeholder="$t('Search discussion')"
                        style="height: 32px"
                        @keydown.enter.prevent="search"
                      />
                      <div class="input-group-append">
                        <ButtonLoading
                          type="btn-sm"
                          mode="button"
                          :loading="searchLoading"
                          :fa-icon="'far'"
                          :icon="'search'"
                          @action="search"
                        ></ButtonLoading>
                      </div>
                    </div>
                  </div>
                </div>
                <div v-if="authorize('discussions', 'create')" class="ml-20-px">
                  <button
                    slot="button"
                    type="button"
                    class="btn btn-primary btn-sm"
                    @click="createDiscussionOpen = !createDiscussionOpen"
                  >
                    {{ $t('Create a Discussion') }}
                  </button>
                </div>
              </div>
            </div>

            <div v-if="createDiscussionOpen" class="mb-30-px">
              <CommentEditor
                :company-slug="$route.params.companySlug"
                :project-slug="$route.params.projectSlug"
                @cancel="cancel"
                @text="created"></CommentEditor>
            </div>

            <b-table class="table-discussions" hover="" :items="comments" :fields="fields" >
              <template v-slot:cell(comment)="data" >
                <router-link
                  :to="{
                    name: 'projects.discussions.show',
                    params: {
                      companySlug: $route.params.companySlug,
                      discussionId: data.item.id,
                    },
                  }" class="txt-primary-title">
                  {{ data.item.comment | truncate(420) }}
                </router-link>
                <div class="stats d-flex justify-content-start">
                  <div class="box-useravatar mr-2 pr-2">
                    <ListUsers :user="data.item.user" :link="true" size="14"></ListUsers>
                    <router-link
                    :to="{
                      name: 'profile.user',
                      params: { username: data.item.user.username },
                    }"
                    >
                      {{data.item.user.name}}
                    </router-link>
                  </div>
                  <div>
                    <span>
                      {{ $t('Created at') }}
                      {{ data.item.created_at.date_for_humans }}
                    </span>
                    /
                    <span >
                      {{ $tc('Last reply at') }}
                      {{ data.item.updated_at.date_for_humans }}
                    </span>
                  </div>
                  <div class="ml-2 pl-2">
                    <font-awesome-icon :icon="['far', 'comments-alt']" />
                    {{ data.item.replies.length }}
                  </div>
                </div>
              </template>
            </b-table>

            <Pagination 
              :total-pages="totalPages" 
              :page="currentPage" 
              :total-rows="totalRows" 
              :per-page="perPage" 
              @change="getDiscussions"></Pagination>

          </div>
        </div>
      </div>
    </div>
  </Layout>
</template>
