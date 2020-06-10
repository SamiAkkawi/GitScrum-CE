<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import TaskFormEnable from '@components/projects/addons/task-form-enable'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import Pagination from '@components/utils/pagination'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'Form2Task',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, ButtonLoading, TaskFormEnable, Pagination },
  data() {
    return {
      dates: '',
      loading: true,
      btnConvertLoading: false,
      answers: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,

      workflow: [],
      workflows: [],
      configFormAnswer: null,

      fields: [
        {
          key: 'title',
          label: this.$t('Answer'),
        },
        {
          key: 'convert',
          label: this.$t('Convert to Task'),
        }
      ],
    }
  },
  created() {
    this.getAnswers(this.currentPage)
    this.getConfig()
  },
  methods: {
    ...modalManager,

    modalTask(value, object) {
      this.open({ name: value, object: object })
    },

    getAnswers(page) {
      this.gotoScrollTop()
      this.loading = true
      Axios()
      .get(
        'form-answers/?company_slug=' + 
        this.$route.params.companySlug + 
        '&project_slug=' + 
        this.$route.params.projectSlug +
        '&page=' +
        page
      )
      .then((response) => {
        if (response.data.data.length === 0 && page !== 1) {
          this.getAnswers(1)
          return
        }
        this.loading = false
        this.answers = response.data.data
        this.totalRows = response.data.total
        this.totalPages = response.data.total_pages
        this.perPage = response.data.per_page
        this.currentPage = response.data.current_page
      })
    },
    getConfig() {
      Axios()
        .get(
          'config-form-answers/?company_slug=' + 
          this.$route.params.companySlug + 
          '&project_slug=' + 
          this.$route.params.projectSlug
        )
        .then((response) => {

          let data = response.data.data
          this.configFormAnswer = data
          this.workflow = data.config_workflow_id
          this.getWorkflows()
          
        })
    },
    getWorkflows() {
      Axios()
        .get(
          'projects-workflows/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          this.workflows = response.data.data
          
          if ( !this.workflow ) {
          
            this.workflows.forEach(element => {
              if ( element.default ){
                this.workflow = element.id
                return
              }
            });

          }
          
        })
    },
    convertToTask(item, index){
      this.btnConvertLoading = true
      Axios()
      .post(
        'form-answers/' + item.uuid + '/convert/?company_slug=' + 
        this.$route.params.companySlug + 
        '&project_slug=' + 
        this.$route.params.projectSlug,
        {
          workflow_id: this.workflow
        }
      )
      .then((response) => {
        this.btnConvertLoading = false
        this.getAnswers();
      })

    },
    updateWorkflow(){
      Axios()
        .put(
          'config-form-answers/workflow/?company_slug=' + 
          this.$route.params.companySlug + 
          '&project_slug=' + 
          this.$route.params.projectSlug, {
            config_workflow_id: this.workflow
          }
        )
        .then((response) => {
          this.btnLoading = false
        })
    },
    remove(item, index){
      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to remove?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.loading = true
          Axios()
          .delete(
            'form-answers/' + item.uuid + '/?company_slug=' + 
            this.$route.params.companySlug + 
            '&project_slug=' + 
            this.$route.params.projectSlug
          )
          .then((response) => {
            this.getAnswers();
          })

        }
      })
    },
    configFormAnswerConfig(status){
      this.configFormAnswer.enabled = status
    }
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading :title="$t('Form2Task')" :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="form2task pt-10px">
      <b-container>
        <b-row>
          <b-col>
            <TaskFormEnable 
              v-if="configFormAnswer" 
              goto="settings"
              :config="configFormAnswer" 
              @enableShareableLink="configFormAnswerConfig"></TaskFormEnable>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-card>
              <div class="d-flex align-items-center">
                <div>{{ $t('Convert to workflow stage') }}</div>
                <div class="ml-10-px wd-150-px">
                  <b-form-select v-model="workflow" 
                  :options="workflows" 
                  value-field="id" 
                  text-field="title" 
                  ize="sm" 
                  @change="updateWorkflow"></b-form-select>
                </div>
              </div>
            </b-card>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-table class="table-time-tracking" hover :items="answers" :fields="fields" >
              <template v-slot:cell(title)="data" >
                <div class="d-flex align-items-center">
                  <span v-if="data.item.status.id === 0" class="badge badge-warning">{{ data.item.status.name}}</span>
                  <span v-if="data.item.status.id === 1" class="badge badge-light">{{ data.item.status.name}}</span>
                  <div v-if="data.item.task && data.item.status.id === 1" >
                    <b-link v-if="data.item.task.code" href="#" class="txt-primary-title txt-link" @click="modalTask('task', data.item.task)">{{ data.item.task.code }} - {{ data.item.task.title }}</b-link >
                    <b-link v-if="!data.item.task.code" href="#" class="txt-primary-title txt-link" @click="modalTask('task', data.item.task)">{{ data.item.task.title }}</b-link>
                  </div>
                  <span v-show="data.item.status.id === 0" class="txt-primary-title" @click="modalAnswer('answer', data.item)">{{ data.item.title}}</span>
                </div>
                <span class="d-block">{{ data.item.description | truncate(240) }}</span>
                <span class="date-time">{{ $t('Created at') }} {{ data.item.created_at.date_for_humans}}</span>
              </template>
              <template v-slot:cell(convert)="data" >
                <div v-if="data.item.status.id === 0">
                  <ButtonLoading
                    :title="$t('Convert')"
                    type="btn-md"
                    mode="button"
                    @action="convertToTask(data.item, data.index)"></ButtonLoading>

                  <a style="margin-top:5px"
                    href="javascript:;"
                    @click="remove(data.item, data.index)">
                    {{ $t('Remove Answer') }}</a>
                </div>
              </template>
            </b-table>

            <Pagination 
              :total-pages="totalPages" 
              :page="currentPage" 
              :total-rows="totalRows" 
              :per-page="perPage" 
              @change="getAnswers"></Pagination> 

          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>
