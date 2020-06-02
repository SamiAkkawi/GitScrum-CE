<script>
import Layout from '@layouts/tpl-main-project'
import appConfig from '@src/app.config'
import Axios from '@utils/axios'
import Sidebar from '@components/projects/settings/side-bar'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import vue2Dropzone from 'vue2-dropzone'
import 'vue2-dropzone/dist/vue2Dropzone.min.css'
import Mentions from '@components/utils/mentions'
import UploadImage from '@components/utils/upload-image'

export default {
  page: {
    title: 'Project Settings',
    meta: [{ name: 'description', content: '' }],
  },
  components: { 
    Layout, 
    Sidebar,
    TitleLoading, 
    ButtonLoading, 
    vueDropzone: vue2Dropzone,
    Mentions,
    UploadImage 
  },
  data() {
    return {
      loading: true,
      btnLoading: false,

      project: [],
      projectsParents: [],
      projectsStatus: [],
      projectsStatusSelect: 0,
      projectsParentSelect: 0,
      companyName: '',
      projectCode: '',
      projectName: '',
      projectDescription: '',
      projectIsPrivate: false,
      projectSlackWebhook: '',
      projectDiscordWebhook: '',
      projectTaskShowNumber: false,
      projectUserTimer: false,
      projectTaskType: false,
      projectTaskEffort: false,
      projectUserStories: false,
      projectSprints: false,
      showSuccessAlert: false,
      alertMessage: '',
      
      logoOptions: {
        url:
          appConfig.APIBaseURL +
          'projects/' +
          this.$route.params.projectSlug +
          '/logo/?company_slug=' +
          this.$route.params.companySlug,
        http: 'POST',
        thumbnailWidth: 128,
        maxFilesize: 5000,
        headers: { Authorization: localStorage.getItem('ACCESS_TOKEN') },
        cropSize: {
          width: 128,
          height: 128,
        },
        paramName: 'logo_file',
      },

      dropzoneOptionsBackground: {
        url:
          appConfig.APIBaseURL +
          'projects/' +
          this.$route.params.projectSlug +
          '/background/?company_slug=' +
          this.$route.params.companySlug,
        thumbnailWidth: 128,
        maxFilesize: 5000,
        maxFiles: 1,
        acceptedFiles: 'image/*',
        headers: { Authorization: localStorage.getItem('ACCESS_TOKEN') },
        paramName: 'background_file',
      },

    }
  },
  created() {
    this.getProject()
  },
  methods: {
    
    setVModel(){
        this.companyName = this.project.company.name
        this.projectCode = this.project.code
        this.projectName = this.project.pure_name
        this.projectDescription = this.project.description
        this.projectIsPrivate = !this.project.visibility.is_private
        this.projectSlackWebhook = this.project.slack_webhook
        this.projectDiscordWebhook = this.project.discord_webhook
        this.projectTaskShowNumber = this.project.settings.show_number
        this.projectUserTimer = this.project.settings.use_timer
        this.projectTaskType = this.project.settings.task_type
        this.projectTaskEffort = this.project.settings.task_effort
        this.projectUserStories = this.project.settings.has_user_stories
        this.projectSprints = this.project.settings.has_sprints
        this.projectsStatusSelect = this.project.status.code
        if (this.project.parent) {
          this.projectsParentSelect = this.project.parent.id
        }
    },

    getProject() {

      this.getProjectStatus()
      this.getProjectParent()

      let project = this.$store.getters['project/getProject']

      if ( project !== null ) {
        this.project = project
        this.loading = false
        this.setVModel()
        return 
      }

      Axios()
      .get('projects/' + 
        this.$route.params.projectSlug + 
        '/?company_slug=' + 
        this.$route.params.companySlug)
      .then((response) => {
        this.loading = false
        this.project = response.data.data
        this.$store.dispatch('project/setProject', this.project)

        this.setVModel()
      })
    },


    getProjectParent() {
      Axios()
      .get(
        'projects/parents/?company_slug=' +
          this.$route.params.companySlug +
          '&project_slug=' +
          this.$route.params.projectSlug
      )
      .then((response) => {
        this.projectsParents = response.data.data
      })
    },

    getProjectStatus() {
      Axios()
      .get(
        'projects/statuses/?company_slug=' +
          this.$route.params.companySlug +
          '&project_slug=' +
          this.$route.params.projectSlug
      )
      .then((response) => {
        this.projectsStatus = response.data.data
        this.projectsStatusSelect = this.project.status.code
      })
    },

    updateProject() {
      this.btnLoading = true

      let parentId = this.projectsParentSelect ? this.projectsParentSelect : null

      Axios()
        .put('projects/' + 
          this.$route.params.projectSlug + 
          '/?company_slug=' + 
          this.$route.params.companySlug, 
        {
          code: this.projectCode,
          name: this.projectName,
          description: this.projectDescription,
          is_private: !this.projectIsPrivate,
          parent_id: parentId,
          current_status: this.projectsStatusSelect.id,
          slack_webhook: this.projectSlackWebhook,
          discord_webhook: this.projectDiscordWebhook,
          show_number: this.projectTaskShowNumber,
          use_timer: this.projectUserTimer,
          task_type: this.projectTaskType,
          task_effort: this.projectTaskEffort,
          has_user_stories: this.projectUserStories,
          has_sprints: this.projectSprints,
        })
        .then((response) => {
          
          this.btnLoading = false

          this.$store.dispatch('project/setProject', response.data.data)
          this.$store.dispatch('projectSidebar/setSidebar', null)

          this.$router.push({
            name: 'project.settings.details',
            params: {
              companySlug: this.$route.params.companySlug,
              projectSlug: response.data.data.slug,
            },
          })

        })
    },

    deleteProject() {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete('projects/' + this.$route.params.projectSlug + '/?company_slug=' + this.$route.params.companySlug)
          .then((response) => {
            this.$router.push({
              name: 'workspaces.projects',
            })
          })
        }
      })
    },

    checked(value) {
      return value === true
    },

    updateLogo(logo) {
      if (this.project.logo !== logo && logo !== null) {
        this.project.logo = logo
      }
    },

    afterCompleteBackground(value) {
      this.project.background = value.dataURL
      this.$refs.updateBackgroundDropzone.removeAllFiles()
    },

    updateDescriptionText(text) {
      this.projectDescription = text
    },

    removeLogo(){
      Axios()
      .delete('projects/' + this.$route.params.projectSlug + 
        '/logo/?company_slug=' + 
        this.$route.params.companySlug)
      .then((response) => {
        window.location.reload();
      })
    },

    removeBackground(){
      Axios()
      .delete('projects/' + this.$route.params.projectSlug + 
        '/background/?company_slug=' + 
        this.$route.params.companySlug)
      .then((response) => {
        window.location.reload();
      })
    }
    
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading :title="$t('Project Details')" :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="project-details pt-10px">
      <b-container>
        <b-row>
          <b-col cols="2">
            <SideBar></SideBar>
          </b-col>
          <b-col cols="9" offset-lg="1">
            <b-card>
              <b-row>
                <b-col cols="6">
                  
                  <div class="mb-0 tx-18-px lh-16 fw-500 txt-001737">{{ $t('Project Logo') }}</div>

                  <UploadImage 
                    :size="128"
                    :options="logoOptions" 
                    :image="project.logo" 
                    :btn-title="$t('Upload Project Logo')"
                    @action="updateLogo"></UploadImage>

                  <p class="tx-12-px txt-A7AFB7">{{ $t('Project_Details_Text_2') }}</p>

                </b-col>
                <b-col cols="6">

                  <div class="mb-15-px d-flex justify-content-between">
                    <div class="tx-18-px lh-16 fw-500 txt-001737">{{ $t('Board Background') }}</div>
                    <b-link href="javascript:;" class="d-block tx-12-px" @click="removeBackground">{{ $t('Remove Background') }}</b-link>
                  </div>

                  <vue-dropzone
                    id="updateBackgroundDropzone"
                    ref="updateBackgroundDropzone"
                    class="mt-15-px"
                    :options="dropzoneOptionsBackground"
                    :use-custom-slot="true"
                    @vdropzone-complete="afterCompleteBackground"
                  >
                    <div class="dropzone-custom-content">
                      <h3 class="dropzone-custom-title">{{ $t('Drag and drop to upload') }}</h3>
                      <div class="subtitle">...{{ $t('or click to select a file from your computer') }}</div>
                    </div>
                  </vue-dropzone>

                  <b-img :src="project.background" class="mt-20-px" style="width:100%"></b-img>

                </b-col>
              </b-row>
            </b-card>
            
            <b-card>
              <b-alert v-model="showSuccessAlert" variant="success" class="mg-b-15" dismissible fade>
                {{ alertMessage }}
              </b-alert>
              <b-form-checkbox v-model="projectIsPrivate" :checked="projectIsPrivate">
                {{ $t('Public Project') }}
              </b-form-checkbox>
              <p class="small">{{ $t('Project_Details_Text') }}</p>
              <b-row class="mt-3">
                <b-col cols="8">
                  <b-row>
                    <b-col cols="8">
                      <b-form-group
                        :label="$t('Company Name')"
                        label-for="company-name">
                        <b-form-input v-model="companyName" type="text" disabled trim></b-form-input>
                      </b-form-group>
                    </b-col>
                    <b-col cols="4">
                      <b-form-group
                        :label="$t('Code - e.g. PROJ01')"
                        label-for="project-code">
                        <b-form-input v-model="projectCode" type="text" maxlength="6" trim></b-form-input>
                      </b-form-group>
                    </b-col>
                  </b-row>
                  <b-row>
                    <b-col cols="8">
                      <b-form-group
                        :label="$t('Project Name')"
                        label-for="project-name">
                        <b-form-input v-model="projectName" type="text" trim></b-form-input>
                      </b-form-group>
                    </b-col>
                    <b-col cols="4">
                      <b-form-group
                        :label="$t('Project Status')"
                        label-for="project-status">
                        <b-form-select
                          v-model="projectsStatusSelect"
                          :options="projectsStatus"
                          value-field="id"
                          text-field="label"></b-form-select>
                      </b-form-group>
                    </b-col>
                  </b-row>
                  <b-row>
                    <b-col cols="12">
                      <b-form-group
                      :label="$t('Project Parent')"
                      label-for="project-parent">
                      <b-form-select
                        v-model="projectsParentSelect"
                        :options="projectsParents"
                        value-field="id"
                        text-field="pure_name">
                        <template v-slot:first>
                          <b-form-select-option :value="null"> -- null -- </b-form-select-option>
                        </template>
                        </b-form-select>
                      </b-form-group>
                    </b-col>
                  </b-row>
                  <b-row>
                    <b-col cols="12">
                      <b-form-group
                        :label="$t('Project Goals')"
                        label-for="project-goals">
                        <Mentions
                        ref="mentions"
                        element-type="textarea"
                        :mention-users="true"
                        :content-text="projectDescription"
                        :company-slug="$route.params.companySlug"
                        :project-slug="$route.params.projectSlug"
                        :element-rows="5"
                        @update-text="updateDescriptionText"></Mentions>
                      </b-form-group>
                    </b-col>
                  </b-row>
                </b-col>
                <b-col cols="4" class="border-left">
                  <b-form-group
                    :label="$t('Project Settings')">
                    <b-form-checkbox v-model="projectTaskShowNumber" :checked="checked(projectTaskShowNumber)" class="mt-2 mb-2">
                      {{ $t('Enable Task Show Number') }}
                    </b-form-checkbox>
                    <b-form-checkbox v-model="projectUserTimer" :checked="checked(projectUserTimer)" class="mb-2">
                      {{ $t('Enable Task Timer') }}
                    </b-form-checkbox>
                    <b-form-checkbox v-model="projectTaskType" :checked="checked(projectTaskType)" class="mb-2">
                      {{ $t('Enable Task Type') }}
                    </b-form-checkbox>
                    <b-form-checkbox v-model="projectTaskEffort" :checked="checked(projectTaskEffort)" class="mb-2">
                      {{ $t('Enable Task Effort') }}
                    </b-form-checkbox>
                    <b-form-checkbox v-model="projectUserStories" :checked="checked(projectUserStories)" class="mb-2">
                      {{ $t('Enable User Stories') }}
                    </b-form-checkbox>
                    <b-form-checkbox v-model="projectSprints" :checked="checked(projectSprints)">
                      {{ $t('Enable Sprints') }}
                    </b-form-checkbox>
                  </b-form-group>
                </b-col>
              </b-row>
              <hr>
              <b-row>
                <b-col cols="6">
                  <b-link @click="deleteProject">{{ $t('Delete') }}</b-link>
                </b-col>
                <b-col cols="6" class="text-right">
                  <ButtonLoading
                    :loading="btnLoading"
                    :title="$t('Update Project')"
                    :title-loading="$t('Updating Project')"
                    type="btn-md"
                    mode="button"
                    @action="updateProject"></ButtonLoading>
                </b-col>
              </b-row>
            </b-card>

            <!--
            <b-card>  
              <div class="tx-18-px lh-16 fw-500 txt-001737 mb-10-px">{{ $t('Project Messaging') }}</div>
              <div class="form-row mg-t-15">
                <div class="form-group col-md-12">
                  <label class="tx-12-px txt-68748F mb-5-px">{{ $t('Slack Webhook') }}</label>
                  <input v-model="projectSlackWebhook" type="text" class="form-control mb-10-px" />
                  <label class="tx-12-px txt-68748F mb-5-px">{{ $t('Discord Webhook') }}</label>
                  <input v-model="projectDiscordWebhook" type="text" class="form-control" />
                </div>
              </div>
            </b-card>
            -->

          </b-col>
        </b-row>
      </b-container>
    
    </div>
  </Layout>
</template>