<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import SideBar from '@components/projects/settings/side-bar'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'

export default {
  page: {
    title: 'Integration',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, SideBar, ButtonLoading },
  data() {
    return {
      loading: true,
      slackLoading: false,
      discordLoading: false,
      slack: '',
      discord: '',
      project: []
    }
  },
  created() {
    this.getProject()
  },
  methods: {

    setVModel(){
        this.slack = this.project.slack_webhook
        this.discord = this.project.discord_webhook
    },

    getProject() {

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

    updateProject(object) {
      this.loading = true

      Axios()
        .put('projects/' + 
          this.$route.params.projectSlug + 
          '/?company_slug=' + 
          this.$route.params.companySlug, 
        object)
        .then((response) => { 
          this.loading = false
        })
    },
    
    updateSlack(){
      this.updateProject({slack_webhook: this.slack})
    },
    
    updateDiscord(){
      this.updateProject({discord_webhook: this.discord})
    }

  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading
        :title="$t('Integrations')"
        :subtitle="$t('Connect GitScrum to external applications')"
        :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="project-integration pt-10px">
      <b-container>
        <b-row>
          <b-col cols="3">
            <SideBar></SideBar>
          </b-col>
          <b-col cols="9">
            <b-card :header="$t('Integrations')">
              <b-row>
                <b-col>
                  <div class="d-flex">
                    <b-img thumbnail fluid 
                      src="https://gitscrum-static.s3.amazonaws.com/img/slack.jpg" 
                      style="width:80px" alt="Slack"></b-img>
                    <div class="ml-2 wd-100">
                      <h6>{{ $t('Connect to Slack') }}</h6>
                      <b-input-group>
                        <b-input-group-append class="wd-100">
                          <b-form-input 
                          v-model="slack" 
                          :placeholder="$t('Slack Webhooks')"
                          type="text" 
                          size="sm"></b-form-input>
                          <ButtonLoading
                          type="btn-sm"
                          icon="check"
                          :loading="slackLoading"
                          @action="updateSlack"
                          ></ButtonLoading>
                        </b-input-group-append>
                      </b-input-group>
                      <small class="mt-1"><b-link href="https://api.slack.com/messaging/webhooks" target="_blank">
                      {{ $t('Getting started with Incoming Webhooks') }}</b-link></small>
                    </div>
                  </div>
                </b-col>
              </b-row>
              <b-row class="mt-3">
                <b-col>
                  <div class="d-flex">
                    <b-img thumbnail fluid 
                      src="https://gitscrum-static.s3.amazonaws.com/img/discord.jpg" 
                      style="width:80px" alt="Discord"></b-img>
                    <div class="ml-2 wd-100">
                      <h6>{{ $t('Connect to Discord') }}</h6>
                      <b-input-group>
                        <b-input-group-append class="wd-100">
                          <b-form-input 
                          v-model="discord" 
                          :placeholder="$t('Discord Webhooks')"
                          type="text" 
                          size="sm"></b-form-input>
                          <ButtonLoading
                          type="btn-sm"
                          icon="check"
                          :loading="discordLoading"
                          @action="updateDiscord"
                          ></ButtonLoading>
                        </b-input-group-append>
                      </b-input-group>
                      <small class="mt-1"><b-link href="https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks" target="_blank">
                      {{ $t('Getting started with Incoming Webhooks') }}</b-link></small>
                    </div>
                  </div>
                </b-col>
              </b-row>
            </b-card>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>