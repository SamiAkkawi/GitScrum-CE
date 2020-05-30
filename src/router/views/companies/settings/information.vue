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
        :loading="loading"
      ></TitleLoading>
    </template>

    <div slot="content" class="company-information pt-10px">
      <div class="row mb-30-px">
        <div class="col-md-2">
          <SideBar></SideBar>
        </div>
        <div class="col-md-9 offset-md-1">

          <div class="card">
            <div class="card-body">
                <span class="tx-12-px txt-68748F">{{ $t('Active license for the company') }}</span>
                <h5>{{ currentCompany.stats.plan.title }}</h5>
            </div>
          </div>

          <div class="card">
            <div class="card-body">

              <b-form-group
                :label="$t('Company Name')"
                label-for="company-name"
              >
                <b-form-input id="company-name" v-model="companyName" trim></b-form-input>
              </b-form-group>

              <b-form-group
                :label="$t('Company Description')"
                label-for="company-name"
              >
                <b-form-textarea
                  id="textarea"
                  v-model="companyDescription"
                  :placeholder="$t('Company description')"
                  rows="3"
                  max-rows="6"
                ></b-form-textarea>
              </b-form-group>

              <div class="row mt-2">
                <div class="col-lg">
                  
                  <label>{{ $t('Company Logo') }}</label>

                  <div class="mt-3 logo">
                    <img v-show="logo !== null" :src="logo" />
                    <div class="buttons">
                      <UploadImage 
                        :two-buttons="true" 
                        :options="logoOptions" 
                        :has-image="true" 
                        @get-image="updateLogo"></UploadImage>
                    </div>
                  </div>
                  <div class="mt-2">
                    <small>
                      {{
                        $t(
                          'You can upload a logo image to appear in your projects and email notifications. Must be at least 128x128 pixels.'
                        )
                      }}
                    </small>
                  </div>

                </div>

                <div class="col-lg">
                  <b-form-checkbox v-model="settingsEmail" :checked="settingsEmail">
                    <span>
                      {{ $t('Emails - Personalised Brand') }}
                    </span>
                  </b-form-checkbox>
                  <small >
                    {{ $t('Company logo and description will appear in emails sent.') }}
                  </small>

                  <b-form-checkbox v-model="settingsLabels" :checked="settingsLabels" class="mt-2">
                    <span>
                      {{ $t('Share Project Labels') }}
                    </span>
                  </b-form-checkbox>
                  <small>
                    {{ $t('Project Labels can be used inside other projects.') }}
                  </small>
                </div>
              </div>

              <div class="mt-3 mb-2">
                <hr>
                <div class="d-flex justify-content-between">
                  <div></div>
                  <ButtonLoading
                    :title="$t('Update Details')"
                    :title-loading="$t('Updating Details')"
                    type="btn-md"
                    @action="updateCompany"
                  ></ButtonLoading>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>
    </div>
  </Layout>
</template>
