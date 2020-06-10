<script>
import Layout from '@layouts/tpl-main'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import SideBar from '@components/companies/side-bar'
import Alert from '@components/utils/alert'
import Pagination from '@components/utils/pagination'

export default {
  page: {
    title: 'Company Team',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, Alert, SideBar, Pagination },
  data() {
    return {
      teammates: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      alertMessage: '',
      alertStatus: false,
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
      loading: false,
      gridConfig: {
        style: [
          '',
          'max-width:200px; width:200px;',
          'max-width:60px; width:60px;',
        ],
      },
      currentUserEmail: null,
      fields: [
        {
          key: 'user',
          label: this.$t('Invited'),
        },
        {
          key: 'accepted',
          label: this.$t('Accepted')
        },
        {
          key: 'resend',
          label: this.$t('Resend')
        }
      ],
    }
  },
  mounted() {
    this.getTeammates(this.currentPage)
  },
  methods: {
    scrollToTop() {
      window.scroll({
        top: 0,
        left: 0,
        behavior: 'smooth',
      })
    },

    getTeammates(page) {
      this.scrollToTop()
      this.loading = true
      Axios()
        .get('team-members/invitations/?company_slug=' + this.currentCompany.slug + '&page=' + page)
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getTeammates(1)
            return
          }
          this.currentUserEmail = response.data.currentUserEmail
          this.teammates = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page

          this.getActiveUsers()

          this.loading = false
        })
        .catch((e) => {
          this.loading = false
          // console.error(e)
        })
    },

    deleteInvitation(item) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          
          this.loading = true
          Axios()
          .delete('team-members/invitations/?company_slug=' + this.currentCompany.slug + '&id=' + item.id)
          .then((response) => {
            this.getTeammates(this.currentPage)
            this.alertStatus = true
            this.alertMessage = response.data.data.message
            this.loading = false
          })
            
        }

      })

    },

    resendInvitation(row) {
      this.loading = true
      Axios()
      .get('team-members/invitations/resend?company_slug=' + 
        this.currentCompany.slug + 
        '&id=' + row.id)
      .then((response) => {
        this.alertStatus = true
        this.alertMessage = response.data.data.message
        this.loading = false
      })
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading
        :title="$t('Company Team Members')"
        :subtitle="$t('See the people you have already invited')"
        :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="company-team pt-10px">
      <div class="row mb-30-px">
        <div class="col-md-3">
          <SideBar></SideBar>
        </div>
        <div class="col-md-9">
          
          <div class="card">
            <div class="card-body">
              <router-link :to="{ 'name': 'companies.teams.index' }" class="small">
                  <font-awesome-icon :icon="['far', 'angle-left']" class="mr-5-px" />
                  {{ $t('Go Back Team Members')}}
              </router-link>
            </div>
          </div>

          <Alert :message="alertMessage" :status="alertStatus" class="mt-10-px"></Alert>

           <b-table class="table-team-members-log" striped hover="" :items="teammates" :fields="fields" >
            <template v-slot:cell(user)="data">
              <p class="mb-0" v-text="data.item.name"></p>
              <small class="small" v-text="data.item.email"></small>
            </template>
            <template v-slot:cell(accepted)="data">
              <span 
                v-if="data.item.accepted_at" 
                class="badge badge-light" 
                v-text="data.item.accepted_at"></span>
            </template>
            <template v-slot:cell(resend)="data">
              <b-link
                v-if="data.item.creator.email !== currentUserEmail && data.item.accepted_at === null"
                @click="deleteInvitation(data.item)" >
                <font-awesome-icon :icon="['fal', 'trash-alt']" class="mr-3" />
              </b-link>
              <b-button
                v-if="data.item.creator.email !== currentUserEmail && data.item.accepted_at === null"
                @click="resendInvitation(data.item)" class="btn btn-sm btn-secondary">
                <font-awesome-icon :icon="['fal', 'paper-plane']" class="ml-5-px" />
                  {{ $t('Resend Invite') }}
              </b-button>
            </template>
          </b-table>

          <Pagination 
          :total-pages="totalPages" 
          :page="currentPage" 
          :total-rows="totalRows" 
          :per-page="perPage" 
          @change="getTeammates"></Pagination>
          
        </div>
      </div>
    </div>
  </Layout>
</template>
