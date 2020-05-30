<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'Sprints',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, ListUsers, TitleLoading, ButtonLoading },
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
      gridConfig: {
        style: [
          'max-width: 45px; padding: 13px 10px 15px 20px;',
          'max-width:543px; width:543px;',
          'max-width:127px; width:127px;',
          'max-width:120px; width:120px;',
          'max-width:158px; width:158px;',
          'max-width: 45px; padding-top:2px',
        ],
      },
      searchTerm: '',

      fields: [
        {
          key: 'status',
          label: 'Status',
        },
        {
          key: 'title',
          label: 'Sprint',
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
                  style="height: 32px"
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
            <div class="ml-20-px" v-if="authorize('sprints', 'create')">
              <button slot="button" type="button" class="btn btn-primary btn-sm" @click="modal('sprintCreate')">
                {{ $t('Create a Sprint') }}
              </button>
            </div>
          </div>
        </div>

        <b-table class="table-sprints" striped hover="" :items="sprints" :fields="fields" >
          <template v-slot:cell(status)="data" >
            <span class="badge badge-light" style="text-transform:uppercase;">
              {{ data.item.status.title }}
            </span>
          </template>
          <template v-slot:cell(title)="data" >
            <div>
              <router-link
                :to="{
                  name: 'projects.sprints.show',
                  params: {
                    companySlug: $route.params.companySlug,
                    projectSlug: $route.params.projectSlug,
                    sprintSlug: data.item.slug,
                  },
                }"
                class="txt-primary-title txt-link"
              >
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

        <div v-if="totalPages > 1" class="d-flex justify-content-center mt-4">
          <b-pagination
            v-model="currentPage"
            hide-goto-end-buttons
            class="paginator"
            :total-rows="totalRows"
            :per-page="perPage"
            @change="getSprints"
          >
            <template slot="prev-text">
              <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
            </template>
            <template slot="next-text">
              <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
            </template>
          </b-pagination>
        </div>
      </div>
    </div>
  </Layout>
</template>
