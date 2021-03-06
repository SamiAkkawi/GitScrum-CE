<script>
import Layout from '@layouts/tpl-main'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import SideBar from '@components/companies/side-bar'
import Alert from '@components/utils/alert'
import Ads from '@components/utils/ads'
import Pagination from '@components/utils/pagination'

export default {
  page: {
    title: 'Billing',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, SideBar, Alert, Ads, Pagination },
  data() {
    return {
      invoices: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      alertMessage: '',
      alertStatus: false,
      isCanceled: null,
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
      loading: false,
      fields: [
        { key: 'document_id', label: 'Document' },
        { key: 'document_url', label: 'Download' },
        { key: 'name', label: 'Subscription' },
        { key: 'period.name', label: 'Period' },
        { key: 'date', label: '' }
      ],
    }
  },
  mounted() {
    this.getSubscriptions(this.currentPage)
    this.getIsCanceled()
  },
  methods: {
    scrollToTop() {
      window.scroll({
        top: 0,
        left: 0,
        behavior: 'smooth',
      })
    },

    getSubscriptions(page) {
      this.scrollToTop()
      this.loading = true
      Axios()
        .get('invoices/?company_slug=' + this.currentCompany.slug + '&page=' + page)
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getSubscriptions(1)
            return
          }
          this.invoices = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },

    getIsCanceled() {
      Axios()
        .get('subscriptions/canceled')
        .then((response) => {
          this.isCanceled = response.data.data
        })
        .catch((e) => {
          console.error(e)
        })
    },

    cancelSubscriptions() {
      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to cancel your subscription?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){

          Axios()
          .delete('subscription/?company_slug=' + this.currentCompany.slug)
          .then((response) => {
            location.reload();
          })
          .catch((e) => {
            console.error(e)
          })

        }
      })
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading
        :title="$t('Billing')"
        :loading="loading"
      ></TitleLoading>
    </template>

    <div slot="content" class="company-team pt-10px">

      <div class="row mb-30-px">
        <div class="col-md-3">
          <SideBar></SideBar>
        </div>
        <div class="col-md-9">
          
          <div v-show="!totalRows">
            <Ads type="large"></Ads>
          </div>

          <div class="card mt-2">
            <div class="card-body d-flex">
              <div class="p-2 flex-fill bd-highlight mr-15-px p-3">
                <span class="d-block tx-18-px fw-600">{{ currentCompany.stats.plan.title }}</span>
                <span v-if="currentCompany.stats.plan.users === 'Unlimited'" class="txt-12-px">
                  {{ $t('Unlimited users') }}
                </span>
                <span v-else  class="txt-12-px">
                  {{ $t('up to users', { users: currentCompany.stats.plan.users }) }}
                </span>
              </div>
              <div v-if="currentCompany.stats.plan.interval === 'monthly' && currentCompany.stats.plan.plan !== ''" class="divTableRowBg p-2 flex-fill bd-highlight p-3 text-right">
                <a v-if="!isCanceled" href="javascript:;" class="tx-12-px" @click="cancelSubscriptions">{{ $t('Cancel Subscription') }}</a>
                <span v-else>{{ $t('Subscription must be canceled at') }}<span class="d-block tx-12-px fw-600">{{ isCanceled }}</span></span>
              </div>
            </div>
          </div>

          <Alert v-if="!invoices.length" :message="$t('You do not have an invoice')" :status="!totalRows" class="mg-b-20"></Alert>

          <b-table class="table-small" hover :items="invoices" :fields="fields" >
            <template v-slot:cell(document_url)="data">
              <div class="d-flex align-items-center">
                <b-link :href="data.item.document_url.data.url" target="_blank">
                  {{ $t('Invoice Download') }}
                </b-link>
              </div>
            </template>
          </b-table>

          <Pagination 
          :total-pages="totalPages" 
          :page="currentPage" 
          :total-rows="totalRows" 
          :per-page="perPage" 
          @change="getSubscriptions"></Pagination>
          
        </div>
      </div>
    </div>
  </Layout>
</template>
