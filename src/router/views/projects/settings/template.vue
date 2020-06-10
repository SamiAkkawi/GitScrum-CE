<script>
import Axios from '@utils/axios'
import Layout from '@layouts/tpl-main-project'
import SideBar from '@components/projects/settings/side-bar'
import TitleLoading from '@components/utils/title-loading'
import TemplateSelected from '@components/companies/templates/selected'
import MessageTemplate from '@components/companies/templates/message'
import ButtonLoading from '@components/utils/button-loading'

export default {
  page: {
    title: 'Workflow Template',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TemplateSelected, SideBar, TitleLoading, MessageTemplate, ButtonLoading },
  data() {
    return {
      loading: true,
      templateLoading: false,
      type: '',
      templates: [],
      templateSelected: null,
      companyTemplates: [],
      companyTemplate: null,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
    }
  },
  created() {
    this.templateType = this.$route.params.template
    this.getTemplate()
    this.listTemplates()
  },
  methods: {
    getTemplate() {
      this.loading = true
      Axios()
        .get('project-templates/' + 
          this.templateType + 
          '/?company_slug=' + 
          this.currentCompany.slug +
          '&project_slug=' +
          this.$route.params.projectSlug)
        .then((response) => {
          this.loading = false
          this.templates = response.data.data
          this.templateSelected = this.templates
        })
    },

    listTemplates() {
      Axios()
        .get('templates/' + 
          this.templateType + 
          '/?company_slug=' + 
          this.currentCompany.slug)
        .then((response) => {
          this.loading = false
          this.companyTemplates = response.data.data
          this.companyTemplate = this.companyTemplates[0].id
        })
    },

    remove(templateSelected){
      this.templates.splice(this.templates.indexOf(templateSelected), 1)
      this.templateSelected = {}
    },

    update() {
      this.getTemplate()
    },

    applyTemplate(){
      this.templateLoading = true
      Axios()
      .post('templates/' + 
        this.templateType + 
        '/' + 
        this.companyTemplate + 
        '/apply/?company_slug=' + 
        this.currentCompany.slug +
        '&project_slug=' +
        this.$route.params.projectSlug)
      .then((response) => {
        if ( response.message ){
          alert(response.message)

        }
        this.getTemplate()
        this.templateLoading = false
      })
    }
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading
        :title="$t('Templates')"
        :loading="loading"
      ></TitleLoading>
    </template>
    <div slot="content" class="template pt-10px">
      <b-container>
        <b-row>
          <b-col cols="2">
            <SideBar></SideBar>
          </b-col>
          <b-col cols="9" offset-lg="1">
            <b-row>
              <b-col cols="12">
                <MessageTemplate :template="templateType"></MessageTemplate>
                <b-card :header="$t('Choose Template')">
                  <b-input-group v-if="companyTemplates[0]">
                    <b-input-group-append class="wd-100">
                      <b-form-select 
                      v-model="companyTemplate" 
                      :options="companyTemplates"
                      :disabled="templateLoading"
                      size="sm" 
                      value-field="id" text-field="name"></b-form-select>
                      <ButtonLoading
                      :loading="templateLoading"
                      type="btn-sm"
                      @action="applyTemplate"></ButtonLoading>
                    </b-input-group-append>
                  </b-input-group>
                  <router-link :to="{ name: 'companies.templates', params: { template: templateType } }" class="mt-1 small">
                    {{ $t('Create a new template to your company') }}
                  </router-link>
                </b-card>
                <TemplateSelected
                  v-if="!loading"
                  :template-selected="templateSelected"
                  :component="templateType"
                  @update="update" 
                  @delete="remove"></TemplateSelected>
              </b-col>
            </b-row>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>
