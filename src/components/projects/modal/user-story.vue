<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'

export default {
  components: { ButtonLoading },
  data() {
    return {
      loading: false,
      projectLoading: false,
      userStoryPriorities: [],
      userStoryPriority: [],
      userStory: '',
      additionalInformation: '',
      acceptanceCriteria: '',
      project: [],
      projects: [],
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      errors: [],
    }
  },
  computed: {
    ...modalManager,
  },
  watch: {
    statusModal(object) {
      if (object.item.name === 'userstoryCreate') {
        object.item.name = ''
        this.getProjects()

        this.$refs['modal'].show()
      }
    },
  },
  methods: {
    hideModal() {
      this.closeModal(this.$refs['modal'])
    },

    changeProjects(projectSlug) {
      this.getUserStoryPriorities(projectSlug)
    },

    getProjects() {
      this.projectLoading = true
      Axios()
        .get('projects/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.projects = response.data.data
          if ( this.$route.params.projectSlug ){
            this.project = this.$route.params.projectSlug
            this.getUserStoryPriorities(this.project)
          } else {
            this.project = this.projects[0].slug
            this.getUserStoryPriorities(this.project)
          }
          this.projectLoading = false
        })
    },

    getUserStoryPriorities(projectSlug) {
      Axios()
        .get('user-story-priorities/?company_slug=' + 
          this.currentCompany.slug + 
          '&project_slug=' + 
          projectSlug)
        .then((response) => {
          this.userStoryPriorities = response.data.data
          this.userStoryPriority = response.data.data[0].id
        })
    },

    updateAdditionalInformationText(text) {
      this.additionalInformation = text
    },

    updateTitleText(text) {
      this.userStory = text
    },

    validateForm(event) {
      this.errors = []

      if (!this.userStory) {
        this.errors.push(this.$t('User Story is required'))
      }

      if (this.errors.length) {
        return false
      }

      return true
    },

    create(event) {
      if (!this.validateForm(event)) {
        event.preventDefault()
        return
      }

      this.loading = true

      Axios()
        .post('user-stories/?company_slug=' + this.currentCompany.slug, {
          project_slug: this.project,
          title: this.userStory,
          user_story_priority_id: this.userStoryPriority,
          additional_information: this.additionalInformation,
          acceptance_criteria: this.acceptanceCriteria,
        })
        .then((response) => {
          let data = response.data
          this.loading = false
          this.$router.push({
            name: 'projects.user-stories.assign-tasks',
            params: {
              companySlug: data.company.slug,
              projectSlug: data.project.slug,
              userStorySlug: data.slug,
            },
          })
        })
        .catch((e) => {
          console.error(e)
        })
    },
  },
}
</script>

<template>
  <b-modal id="modal" ref="modal" hide-header hide-footer>
    <b-container>
      <b-row class="mb-3">
        <b-col>
          <h3 class="mb-0">{{ $t('Create an User Story') }}</h3>
          <span>{{ currentCompany.name }}</span>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Project Name')">
            <b-form-select 
            v-model="project" 
            :options="projects" 
            value-field="slug"
            text-field="name"
            @change="changeProjects"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('User Story Priority')">
            <b-form-select 
              v-model="userStoryPriority" 
              :options="userStoryPriorities"
              :disabled="projectLoading"
              value-field="id"
              text-field="title"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('User Story')">
            <b-form-textarea
              v-model="userStory"
              :disabled="projectLoading"
              :placeholder="$t('As a [persona] I want to [do something/get something] so then I can [get something/accomplish something]')"
              rows="4"
              max-rows="6"></b-form-textarea>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Additional Information')">
            <b-form-textarea
              v-model="additionalInformation"
              :disabled="projectLoading"
              rows="1"
              max-rows="6"></b-form-textarea>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Acceptance Criteria')">
            <b-form-textarea 
              v-model="acceptanceCriteria" 
              :disabled="projectLoading"
              rows="1" 
              max-rows="4"></b-form-textarea>
            <small class="mt-2">{{ $t('UserStory_Text3') }}</small>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col align-self="center">
          <b-link 
            v-show="!loading" 
            @click="hideModal" 
            v-text="$t('Cancel')" />
        </b-col>
        <b-col class="text-right">
          <ButtonLoading
            :loading="loading"
            :title="$t('Create User Story')"
            :title-loading="$t('Creating')"
            type="btn-md"
            mode="button"
            @action="create"></ButtonLoading>
        </b-col>
      </b-row>
    </b-container>
  </b-modal>
</template>
