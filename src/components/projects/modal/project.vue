<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { modalManager } from '@state/helpers'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css'
import { faTrumpet } from '@fortawesome/pro-regular-svg-icons'

export default {
  components: { vSelect, ButtonLoading },
  data() {
    return {
      loading: false,
      projectsStatus: [],
      projectsStatusSelect: 0,
      projectsParents: [],
      projectsParentSelect: 0,
      projectName: '',
      projectCode: '',
      projectDescription: '',
      projectIsPrivate: 1,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      errors: [],

      advancedShow: false
    }
  },
  computed: {
    ...modalManager,
  },
  watch: {
    projectIsPrivate(){
      
    },
    statusModal(object) {
      if (object.item.name === 'projectCreate') {
        object.item.name = ''
        this.projectsStatus = []
        this.projectsStatusSelect = 0
        this.projectsParents = []
        this.projectsParentSelect = 0
        this.projectName = ''
        this.projectCode = ''
        this.projectDescription = ''
        this.projectIsPrivate = 1
        this.errors = []
        this.getProjectStatus()
        this.getProjectParent()
        this.$refs['modal'].show()
      }
    },
  },
  methods: {
    hideModal() {
      this.closeModal(this.$refs['modal'])
    },

    getProjectParent() {
      Axios()
        .get('projects/parents/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          let projects = response.data.data

          if ( this.$route.params.projectSlug ){
            let arr = []
            for (var i = 0; i < projects.length; i++) {
              if ( this.$route.params.projectSlug !== projects[i].slug ){
                arr[i] = projects[i]
              }
            }
            this.projectsParents = arr
          } else {
            this.projectsParents = projects
          }
          
        })
    },

    getProjectStatus() {
      Axios()
        .get('projects/statuses/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.projectsStatus = response.data.data
          this.projectsStatusSelect = this.projectsStatus[0].id
        })
    },

    create(event) {

      this.loading = faTrumpet

      let parentId = this.projectsParentSelect ? this.projectsParentSelect.id : null
      
      Axios()
        .post('projects/?company_slug=' + this.currentCompany.slug, {
          company_slug: this.currentCompany.slug,
          name: this.projectName,
          code: this.projectCode,
          description: this.projectDescription,
          parent_id: parentId,
          current_status: this.projectsStatusSelect.id,
          is_private: this.projectIsPrivate,
        })
        .then((response) => {
          let data = response.data.data
          this.loading = false
          this.$router.push({
            name: 'projects.board',
            params: {
              companySlug: data.company.slug,
              projectSlug: data.slug,
            },
          })
        })
    },

    advanced(){
      this.advancedShow = !this.advancedShow
    }
  },
}
</script>

<template>
  <b-modal id="modal" ref="modal" hide-header hide-footer>

    <b-container>
      <b-row class="mb-3">
        <b-col>
          <h3 class="mb-0">{{ $t('Create a Project') }}</h3>
          <span>{{ currentCompany.name }}</span>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('Project Name')">
            <b-form-input 
            v-model="projectName" 
            type="text" 
            maxlength="45" trim></b-form-input>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row>
        <b-col class="d-flex justify-content-between mb-2">
          <b-form-checkbox
            v-model="projectIsPrivate"
            value="1"
            unchecked-value="0">
            {{ $t('Project Private') }}
          </b-form-checkbox>
          <b-link class="link-advanced" @click="advanced">{{ $t('Advanced') }}</b-link>
        </b-col>
      </b-row>
      <b-row v-show="advancedShow" class="mt-3">
        <b-col>
          <b-form-group
            :label="$t('Optional Code (e.g. PROJ01)')">
            <b-form-input 
            v-model="projectCode"
            type="text" 
            maxlength="6" 
            trim></b-form-input>
          </b-form-group>
        </b-col>
        <b-col>
          <b-form-group
            :label="$t('Project Status')">
            <b-form-select 
            v-model="projectsStatusSelect" 
            :options="projectsStatus"
            value-field="id"
            text-field="label"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row v-show="advancedShow">
        <b-col>
          <b-form-group
            :label="$t('Project Parent')">
            <b-form-select 
            v-model="projectsParentSelect" 
            :options="projectsParents"
            value-field="slug"
            text-field="name"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row v-show="advancedShow">
        <b-col>
          <b-form-group
            :label="$t('Project Goals')">
            <b-form-textarea 
            v-model="projectDescription" 
            rows="2" 
            max-rows="4"></b-form-textarea>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row class="mt-2">
        <b-col align-self="center">
          <b-link 
            v-show="!loading" 
            @click="hideModal" 
            v-text="$t('Cancel')" />
        </b-col>
        <b-col class="text-right">
          <ButtonLoading
            :loading="loading"
            :title="$t('Create Project')"
            :title-loading="$t('Creating')"
            type="btn-md"
            mode="button"
            @action="create"></ButtonLoading>
        </b-col>
      </b-row>
    </b-container>
  </b-modal>
</template>
