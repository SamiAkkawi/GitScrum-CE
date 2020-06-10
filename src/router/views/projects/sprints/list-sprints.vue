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
    title: 'Sprints',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, ListUsers, TitleLoading, ButtonLoading, Pagination },
  data() {
    return {
      loading: false,
      searchLoading: false,
      seachByProjectName: '',
      sprints: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      searchTerm: '',

      fields: [
        {
          key: 'status',
          label: this.$t('Status'),
        },
        {
          key: 'title',
          label: this.$t('Sprint'),
        },
      ],
    }
  },
  mounted() {
    this.getSprints(this.currentPage)
  },
  methods: {
    ...modalManager,

    modal(value) {
      this.open({ name: value, object: [] })
    },

    getSprints(page) {
      this.gotoScrollTop()
      this.loading = true
      Axios()
        .get(
          'sprints/?company_slug=' +
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
            this.getSprints(1)
            return
          }
          this.sprints = response.data.data
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
      this.sprints = []
      this.currentPage = 1
      this.getSprints(1)
      this.loading = false
    },
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading
        :title="$tc('Explore Sprints', totalRows)"
        :title-alternative="$t('Sprints')"
        :subtitle="$t('Sprint is one timeboxed iteration of a continuous cycle')"
        :loading="loading"
      ></TitleLoading>
    </template>

    <div slot="content" class="sprint pt-10px">
      
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
                  :placeholder="$t('Search sprint')"
                  @keydown.enter.prevent="search"
                  style="height: 32px" />
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
            <div v-if="authorize('sprints', 'create')" class="ml-20-px">
              <button slot="button" type="button" class="btn btn-primary btn-sm" @click="modal('sprintCreate')">
                {{ $t('Create a Sprint') }}
              </button>
            </div>
          </div>
        </div>

        <b-table class="table-sprints" hover :items="sprints" :fields="fields" >
          <template v-slot:cell(status)="data" >
            <span class="badge badge-light text-uppercase">
              {{ data.item.status.title }}
            </span>
          </template>
          <template v-slot:cell(title)="data" >
            <div>
              <router-link
                :to="{ name: 'projects.sprints.show',
                params: {
                  companySlug: $route.params.companySlug,
                  projectSlug: $route.params.projectSlug,
                  sprintSlug: data.item.slug } }" class="txt-primary-title txt-link">
                {{ data.item.code }} - {{ data.item.title }}
              </router-link>
              <p class="mb-0">
                <span>{{ $t('Completed') }}: {{ parseFloat(data.item.stats.completed) | percent(0)  }}</span>
                -
                <span>{{ $t('Timebox') }}: {{ data.item.timebox }}</span>
                -
                <span> {{ $t('Duration') }}: {{ data.item.duration }} </span>
              </p>
            </div>
            <div class="mt-1 d-flex align-items-center justify-content-start">
              <ListUsers :users="data.item.users" :link="true" :limit="20" :wrap="true" :size="22"></ListUsers>
            </div>
          </template>
        </b-table>

        <Pagination 
          :total-pages="totalPages" 
          :page="currentPage" 
          :total-rows="totalRows" 
          :per-page="perPage" 
          @change="getSprints"></Pagination>

      </div>
    </div>
  </Layout>
</template>
