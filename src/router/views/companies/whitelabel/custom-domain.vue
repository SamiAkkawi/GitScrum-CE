<script>
import Layout from '@layouts/tpl-main'
import appConfig from '@src/app.config'
import SideBar from '@components/companies/side-bar'
import TitleLoading from '@components/utils/title-loading'
import Axios from '@utils/axios'
import UploadImage from '@components/utils/upload-image'
import Alert from '@components/utils/alert'
import ButtonLoading from '@components/utils/button-loading'
import Ads from '@components/utils/ads'

export default {
  page: {
    title: 'White Label Setup',
    meta: [{ name: 'description', content: 'White Label' }],
  },
  components: { Layout, TitleLoading, UploadImage, SideBar, Alert, Ads, ButtonLoading },
  data: function() {
    return {
      loading: false,
      loadingEmail: true,
      loadingAddDomain: false,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      whiteLabel: null,
      logoOptions: null,
      emailFooter: null,
      domains: [],
      newDomain: '',
      logo: null,
      alertMessage: '',
      alertStatus: false,
    }
  },
  created() {
    this.getCompany()
    this.getEmail()
    this.getDomains()
    this.getLogo()
    this.logoOptions = this.setLogoOptions()
  },
  methods: {
    getCompany() {
      Axios()
        .get('companies/' + this.currentCompany.slug)
        .then((response) => {
          this.whiteLabel = response.data.data.whitelabel.enabled
        })
    },

    setLogoOptions() {
      return {
        url: appConfig.APIBaseURL + 'whitelabel/logo/?company_slug=' + this.currentCompany.slug,
        http: 'POST',
        thumbnailWidth: 122,
        maxFilesize: 5000,
        headers: { Authorization: localStorage.getItem('ACCESS_TOKEN') },
        paramName: 'logo_file',
        cropSize: {
          width: 122,
          height: 22,
        },
      }
    },
    getEmail() {
      Axios()
        .get('whitelabel/email/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.emailFooter = response.data.data.email
          this.loadingEmail = false
        })
        .catch((e) => {
          console.error(e)
        })
    },

    addEmail() {
      this.loadingEmail = true
      Axios()
        .put('whitelabel/email/', {
          company_slug: this.currentCompany.slug,
          emailFooter: this.emailFooter,
        })
        .then((_) => {
          this.loadingEmail = false
        })
        .catch((e) => {
          console.error(e)
          this.loadingEmail = false
        })
    },

    getDomains() {
      Axios()
        .get('whitelabel/domains/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.domains = response.data.data
        })
        .catch((error) => {
          console.error(error)
        })
    },

    addDomain() {
      this.loadingAddDomain = true
      this.alertStatus = false
      Axios()
        .post('whitelabel/domains/', {
          company_slug: this.currentCompany.slug,
          domain_name: this.newDomain,
        })
        .then((response) => {
          const domain = this.newDomain
          this.domains.push({
            domain: domain,
          })
          this.newDomain = ''
          this.loadingAddDomain = false
        })
        .catch((error) => {
          this.alertMessage = 'Error. '

          error.response.data.data.domain_name.forEach((message) => {
            this.alertMessage += message
          })

          this.alertStatus = true
          this.loadingAddDomain = false
        })
    },

    deleteDomain(domainName) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete('whitelabel/domains/?company_slug=' + this.currentCompany.slug + '&domain_name=' + domainName)
          .then((response) => {
            let domainPosition = this.domains
              .map(function(e) {
                return e.domain
              })
              .indexOf(domainName)
            this.domains.splice(domainPosition, 1)
          })
          .catch((e) => {
            console.error(e)
          })
        }

      })
        
    },

    getLogo() {
      Axios()
        .get('whitelabel/logo/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.logo = response.data.data.logo
        })
        .catch((e) => {
          console.error(e)
        })
    },

    removeLogo() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete('whitelabel/logo/?company_slug=' + this.currentCompany.slug)
          .then((response) => {
            this.logo = null
            let domain = window.location.host
            this.$cookies.remove('branding-' + domain)
          })
          .catch((e) => {
            console.error(e)
          })
        }

      })

    },

    updateLogo(logo) {
      if (this.logo !== logo && logo !== null) {
        document.getElementById("companyLogo").src= logo;
        this.$store.dispatch('whiteLabel/setWhiteLabel', null)
        this.logo = logo
      }
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading 
        :title="$t('White Label Setup')" 
        :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="white-label pt-10px">
      <b-row>
        <b-col cols="3"><SideBar></SideBar></b-col>
        <b-col cols="5">
          <b-card :header="$t('STEP 1')">
            <p>{{ $t('Add a Domain / Subdomain ( Mandatory )') }}</p>
            <Alert :message="alertMessage" :status="alertStatus"></Alert>
            <small v-show="!whiteLabel">{{ $t('You must update your account to set up White Label') }}</small>
            <b-input-group v-show="whiteLabel" class="mb-1">
              <b-input-group-append>
                <b-form-input 
                v-model="newDomain" 
                :placeholder="$t('e.g.: my-domain.com / project.my-domain.com')"
                type="text" 
                autocomplete="off"
                size="sm"></b-form-input>
                <ButtonLoading
                :loading="loading"
                type="btn-sm"
                icon="plus"
                @action="addDomain"
                ></ButtonLoading>
              </b-input-group-append>
            </b-input-group>
            <small>* {{ $t('Domains and/or Subdomains that are allowed to use GitScrum White Label.') }}</small>   
            <div v-show="domains">
              <h6 v-if="domains.length > 0" class="mt-15-px">{{ $t('Domains / Subdomains List') }}</h6>
              <table class="table">
                <tbody>
                  <tr v-for="domain in domains" :key="domain.id">
                    <td>
                      <b-link :href="'http://' + domain.domain" target="_blank">{{ domain.domain }}</b-link>
                    </td>
                    <td class="text-right">
                      <b-link @click="deleteDomain(domain.domain)">
                        <font-awesome-icon :icon="['far', 'trash-alt']" />
                      </b-link>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </b-card>
          <b-card :header="$t('STEP 2')">
            <p>{{ $t("Add a CNAME record to your domain's DNS records") }}</p>
            <p>{{$t('To add the CNAME record to your domain host, follow the steps below. See your domain host’s documentation for more specific instructions.') }}
            </p>
            <p> {{ $t('1. Go to your domain’s DNS records') }}. </p>
            <p>{{ $t('2. Add a record to your DNS settings, selecting CNAME as the record type') }}.</p>
            <p>{{ $t('3. Return to the first window or tab and copy the contents of the Label/Host field') }}.</p>
            <p>{{ $t('4. Paste the copied contents into the Label or Host field with your DNS records') }}.</p>
            <p>{{$t('5. Return to the first window or tab and copy the contents of the Destination/Target field')}}.</p>
            <p>{{ $t('6. Paste the copied contents into the Destination or Target field with your DNS records') }}.</p>
            <div class="mt-3">
              <p class="fw-600">{{ $t('Your record should look similar to one of the tables below') }}:</p>
              <p>{{ $t('GitScrum White Label - CNAME') }} : <span class="fw-600">wl.gitscrum.com</span></p>
            </div>
            <div class="mt-2 mb-2 ml-3">
              <p><strong>{{ $t('Record type') }}</strong>: {{ $t('CNAME') }}
              <br><strong>{{ $t('Label/Host field') }}</strong>: XXXXXXXXXXXX
              <br><strong>{{ $t('Time To Live (TTL)') }}</strong>: {{ $t('3600 or leave the default') }}
              <br><strong>{{ $t('Destination/Target field') }}</strong>: wl.gitscrum.com</p>
            </div>
            <p>{{ $t('7. Save your record') }}.</p>
            <p>{{ $t('CNAME_Text1') }}</p>
          </b-card>
        </b-col>
        <b-col cols="4">
          <Ads v-show="!whiteLabel" type="square"></Ads>
          <b-card :header="$t('Company Logo')">
            <b-card-text>
              <p>{{ $t('You can upload a logo to replace the GitScrum Logo in Header of your company. This is a great way to promote your brand.') }}</p>
              <p>{{ $t('We recommend using image files: of less than 500 KB and 122 (width) x 22 (height) pixels for best results.') }}</p>
            </b-card-text>
            <b-card-text v-show="whiteLabel" class="mt-15-px">
              <div v-show="logoOptions">
                <UploadImage 
                  :size="128"
                  :options="logoOptions" 
                  :image="logo" 
                  :btn-title="$t('Upload your Company Logo')"
                  @action="updateLogo"></UploadImage>
              </div>
            </b-card-text>
          </b-card>
          <b-card v-show="whiteLabel" :header="$t('Email Footer')">
            <b-card-text>
              <b-input-group v-show="whiteLabel" class="mb-1">
                <b-input-group-append>
                  <b-form-input 
                  v-model="emailFooter"
                  :placeholder="$t('Write a new email footer')"
                  type="text" 
                  autocomplete="off"
                  size="sm"></b-form-input>
                  <ButtonLoading
                  :loading="loading"
                  type="btn-sm"
                  icon="plus"
                  @action="addEmail"
                  ></ButtonLoading>
                </b-input-group-append>
              </b-input-group>
            </b-card-text>
          </b-card>
        </b-col>
      </b-row>
    </div>
  </Layout>
</template>
