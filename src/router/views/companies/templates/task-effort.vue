<script>
import Axios from '@utils/axios'
import Layout from '@layouts/tpl-main'
import SideBar from '@components/companies/side-bar'
import TitleLoading from '@components/utils/title-loading'
import TemplateTaskEffort from '@components/companies/templates/task-effort'
import CreateTemplate from '@components/companies/templates/create'
import ListTemplate from '@components/companies/templates/list'
import MessageTemplate from '@components/companies/templates/message'

export default {
  page: {
    title: 'Task Effort Template',
    meta: [{ name: 'description', content: 'Task Type' }],
  },
  components: { Layout, TemplateTaskEffort, SideBar, TitleLoading, CreateTemplate, ListTemplate, MessageTemplate },
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
        .get('templates/effort/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          const templateList = this.handleTemplatePrivacy(response.data.data)
          if (!this.templateSelected && templateList[0]) {
            this.templateSelected = templateList[0]
          }

          this.templateList = templateList
          this.loading = false
        })
        .catch((e) => {
          this.loading = false
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

    updateList() {
      this.listTemplates()
    },
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading
        :title="$t('Effort Templates')"
        :loading="loading"
      ></TitleLoading>
    </template>
    <div slot="content" class="template template-workflow pt-10px">
      <b-row>
        <b-col cols="2">
          <SideBar></SideBar>
        </b-col>
        <b-col cols="9" offset-lg="1">
          <MessageTemplate template="effort"></MessageTemplate>
          <b-row>
            <b-col cols="4">
              <CreateTemplate template="effort" @create="createTemplate"></CreateTemplate>
              <ListTemplate v-if="!loading" :templates="templates" @selected="selected"></ListTemplate>
            </b-col>
            <b-col cols="8">
              <TemplateTaskEffort 
                v-if="!loading"
                :template-selected="templateSelected" 
                @update-list="updateList"
                @delete="remove"></TemplateTaskEffort>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </div>
  </Layout>
</template>
