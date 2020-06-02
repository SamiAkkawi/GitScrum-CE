<script>
import Axios from '@utils/axios'
import Layout from '@layouts/tpl-main-project'
import SideBar from '@components/projects/settings/side-bar'
import TitleLoading from '@components/utils/title-loading'
import TemplateSelected from '@components/companies/templates/selected'
import MessageTemplate from '@components/companies/templates/message'

export default {
  page: {
    title: 'Workflow Template',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TemplateSelected, SideBar, TitleLoading, MessageTemplate },
  data() {
    return {
      loading: true,
      loadingCreate: false,
      type: '',
      templates: [],
      templateSelected: null,
      name: '',
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
    }
  },
  created() {
    
    this.templateType = this.$route.params.template

    this.listTemplates()
  },
  methods: {
    listTemplates() {
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

          console.log(this.templates)
          this.templateSelected = this.templates
          
        })
    },

    createTemplate(template) {
      this.templates.push(template)
      this.selected(template)
    },

    selected(template) {
      this.templateSelected = template
    },

    remove(templateSelected){
      this.templates.splice(this.templates.indexOf(templateSelected), 1)
      this.templateSelected = {}
    },

    update() {
      this.listTemplates()
    },
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
