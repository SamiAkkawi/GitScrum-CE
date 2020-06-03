<script>
import Layout from '@layouts/tpl-main'
import appConfig from '@src/app.config'
import SideBar from '@components/companies/side-bar'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import UploadImage from '@components/utils/upload-image'
import ButtonLoading from '@components/utils/button-loading'

export default {
  page: {
    title: 'Company Information',
    meta: [{ name: '', content: '' }],
  },
  components: {
    Layout,
    SideBar,
    TitleLoading,
    UploadImage,
    ButtonLoading,
  },
  data: function() {
    return {
      loading: true,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      company: [],
      companyName: '',
      companyDescription: '',
      companyDescriptionMention: '',
      settingsEmail: false,
      settingsLabels: false,
      btnLoading: false,
      logo: null,

      logoOptions: {
        url: appConfig.APIBaseURL + '/companies/',
        http: 'POST',
        thumbnailWidth: 150,
        maxFilesize: 100,
        headers: { Authorization: localStorage.getItem('ACCESS_TOKEN') },
        paramName: 'logo_file',
      },
      companyLogo: null,
    }
  },
  created() {
    this.getCompany(this.currentCompany.slug)
    this.logoOptions.url += this.currentCompany.slug + '/logo'
  },
  methods: {
    getCompany(companySlug) {
      Axios()
        .get('companies/' + companySlug)
        .then((response) => {
          this.loading = false
          this.company = response.data.data
          this.currentCompany = this.company
          
          this.logo = this.company.logo
          this.companyName = this.company.name
          this.companyDescription = this.company.description
          this.companyDescriptionMention = this.company.description_mention
          this.settingsEmail = this.company.settings.personalised_email
          this.settingsLabels = this.company.settings.share_project_labels

          localStorage.setItem('CURRENT_COMPANY', JSON.stringify(this.company))
          this.$eventBus.$emit('change-current-company', this.company)
        })
        .catch((e) => {
          console.error(e)
        })
    },

    getCompanies(){
      Axios()
        .get('companies')
        .then((response) => {

          localStorage.setItem('COMPANIES_USER', JSON.stringify(response.data.data))

        })
        .catch((e) => {
          console.error(e)
        })
       

    },

    updateCompany() {
      this.loading = true
      
      Axios()
        .put('companies/' + this.currentCompany.slug, {
          name: this.companyName,
          description: this.companyDescription,
          description_mention: this.companyDescriptionMention,
          personalised_email: this.settingsEmail,
          has_company_labels: this.settingsLabels,
        })
        .then((response) => {
          this.loading = false
          this.getCompany(response.data.data.slug)
          this.getCompanies()
        })
        .catch((e) => {
          console.error(e)
        })
    },

    updateLogo(logo) {
      this.logo = logo
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading
        :title="$t('Company Details')"
        :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="company-information pt-10px">
      <b-row>
        <b-col cols="3"><SideBar></SideBar></b-col>
        <b-col cols="5">
          <b-card :header="$t('Company Details')">
            <b-form-group
                :label="$t('Company Name')"
                label-for="company-name">
                <b-form-input id="company-name" v-model="companyName" trim></b-form-input>
              </b-form-group>
              <b-form-group
                :label="$t('Company Description')"
                label-for="company-name">
                <b-form-textarea
                  id="textarea"
                  v-model="companyDescription"
                  :placeholder="$t('Company description')"
                  rows="3"
                  max-rows="6"
                ></b-form-textarea>
              </b-form-group>
              <b-form-checkbox v-model="settingsEmail" :checked="settingsEmail">
                <span> {{ $t('Emails - Personalised Brand') }}</span>
              </b-form-checkbox>
              <small>{{ $t('Company logo and description will appear in emails sent.') }}</small>
              <b-form-checkbox v-model="settingsLabels" :checked="settingsLabels" class="mt-2">
                <span> {{ $t('Share Project Labels') }}</span>
              </b-form-checkbox>
              <small>{{ $t('Project Labels can be used inside other projects.') }}</small>
              <div class="mt-3 mb-2">
                <hr>
                <div class="d-flex justify-content-between">
                  <div></div>
                  <ButtonLoading
                    :title="$t('Update Details')"
                    :title-loading="$t('Updating Details')"
                    :loading="loading"
                    type="btn-md"
                    @action="updateCompany"
                  ></ButtonLoading>
                </div>
              </div>
          </b-card>
        </b-col>
        <b-col cols="4">
          <b-card :header="$t('Company Subscription')">
            <span class="small">{{ $t('Active license for the company') }}</span>
            <h5 class="h6 font-weight-bold">{{ currentCompany.stats.plan.title }}</h5>
          </b-card>
          <b-card :header="$t('Company Logo')">
            <UploadImage 
              :size="128"
              :options="logoOptions" 
              :image="logo" 
              :btn-title="$t('Upload Company Logo')"
              @action="updateLogo"></UploadImage>
              <small> {{$t('You can upload a logo image to appear in your projects and email notifications. Must be at least 128x128 pixels.')}}</small>
          </b-card>
        </b-col>
      </b-row>
    </div>
  </Layout>
</template>
