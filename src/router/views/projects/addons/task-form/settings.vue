<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import TaskFormEnable from '@components/projects/addons/task-form-enable'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'Form2Taskm',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, ButtonLoading,  TaskFormEnable },
  data() {
    return {
      configFormAnswer: null,
      loading: true,
      btnLoading: false,
      enableShareableLinkLoading: false,
      enableShareableLink: false,
      formAnswerTitle: '',
      fieldTitle: '',
      fieldDescription: '',
      token: '',
      shareableLink: '',
      linkCopied: false,
    }
  },
  created() {
    this.getConfigFormAnswers()
  },
  methods: {
    ...modalManager,

    modal(value) {
      this.open({ name: value, object: [] })
    },

    getConfigFormAnswers() {
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
          this.formAnswerTitle = data.main_title
          this.fieldTitle = data.field_title
          this.fieldDescription = data.field_description
          this.loading = false
        })
    },

    formSubmit(){
      this.loading = true
      Axios()
        .put(
        'config-form-answers/?company_slug=' + 
        this.$route.params.companySlug + 
        '&project_slug=' + 
        this.$route.params.projectSlug, {
          main_title: this.formAnswerTitle,
          field_title: this.fieldTitle,
          field_description: this.fieldDescription
        }
        )
        .then((response) => {
          this.loading = false
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
              goto="answers"
              :config="configFormAnswer" 
              @enableShareableLink="configFormAnswerConfig"></TaskFormEnable>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-card :header="$t('Form2Task Fields')">
              <b-form-group :label="$t('Form2Task Title')">
                <b-form-input v-model="formAnswerTitle" :disabled="!configFormAnswer.enabled"></b-form-input>
              </b-form-group>
              <b-form-group :label="$t('Ask a question, the answer will be the task title')">
                <b-form-input v-model="fieldTitle" :disabled="!configFormAnswer.enabled"></b-form-input>
              </b-form-group>
              <b-form-group :label="$t('Ask a question, the answer will be the task description')">
                <b-form-input v-model="fieldDescription" :disabled="!configFormAnswer.enabled"></b-form-input>
              </b-form-group>
              <div v-if="configFormAnswer.enabled">
                <p class="small mt-2">
                  {{ $t('You can add custom fields to the Form2Task.') }} {{ $t('Whenever you modify the custom fields they will appear in the tasks.') }}
                </p>
                <hr>
                <div class="d-flex justify-content-between mt-15-px">
                  <div>
                    <button class="btn btn-secondary" @click="modal('projectTaskCustomFields')">Manage Custom Fields</button>
                  </div>
                  <ButtonLoading
                    :title="$t('Save')"
                    type="btn-md"
                    mode="button"
                    @action="formSubmit"
                  ></ButtonLoading>
                </div>
              </div>
            </b-card>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>
