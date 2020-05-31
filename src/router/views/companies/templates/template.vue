<script>
import Axios from '@utils/axios'
import Layout from '@layouts/tpl-main'
import SideBar from '@components/companies/side-bar'
import TitleLoading from '@components/utils/title-loading'
import TemplateSelected from '@components/companies/templates/selected'
import CreateTemplate from '@components/companies/templates/create'
import ListTemplate from '@components/companies/templates/list'
import MessageTemplate from '@components/companies/templates/message'

export default {
  page: {
    title: 'Workflow Template',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TemplateSelected, SideBar, TitleLoading, CreateTemplate, ListTemplate, MessageTemplate },
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
        .get('templates/' + 
          this.templateType + 
          '/?company_slug=' + 
          this.currentCompany.slug)
        .then((response) => {
          
          this.loading = false
          this.templates = response.data.data
          if ( !this.templateSelected ){
            this.templateSelected = this.templates[0]
          }
          
        })
        .catch((e) => {
          this.loadingCreate = false
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
      <b-row>
        <b-col cols="2">
          <SideBar></SideBar>
        </b-col>
        <b-col cols="9" offset-lg="1">
          <MessageTemplate :template="templateType"></MessageTemplate>
          <b-row>
            <b-col cols="4">
              <CreateTemplate :template="templateType" @create="createTemplate"></CreateTemplate>
              <ListTemplate v-if="!loading" :templates="templates" @selected="selected"></ListTemplate>
            </b-col>
            <b-col cols="8">
              <TemplateSelected
                v-if="!loading"
                :template-selected="templateSelected" 
                :component="templateType"
                @update="update" 
                @delete="remove"> </TemplateSelected>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </div>
  </Layout>
</template>
