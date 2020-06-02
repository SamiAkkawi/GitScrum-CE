<script>
import Axios from '@utils/axios'
import ProjectMenuSettings from '@layouts/partials/project-menu-settings'
import ProjectVisibility from '@components/projects/project-visibility'
import ListUsers from '@components/utils/list-users'
import ProjectBoardFilters from '@layouts/partials/header-board-filters'
import ProjectIntegrations from '@components/projects/modal/integrations/index'
import { isMobile } from 'mobile-device-detect';
import { modalManager } from '@state/helpers'
import { SidebarMenu } from 'vue-sidebar-menu'

export default {
  components: {
    ProjectMenuSettings,
    ProjectVisibility,
    ListUsers,
    ProjectBoardFilters,
    ProjectIntegrations,
    SidebarMenu,
  },
  props: {
    background: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      isMobile: isMobile,
      project: [],
      menu: [],
      
      sidebarStatus: false,
      projectExtra: false,
      projectExtraIcon: 'minus-square'
    }
  },
  
  mounted(){
    this.$store.dispatch('tasksArchived/setIsArchived', this.btnOptionSelected)
    let state = this.getMenuProjectState()
    this.onToggleCollapse(state)
    this.setBackground()
  },
  created() {
    this.getProject()
  },
  methods: {
    ...modalManager,

    modal(value) {
      this.open({ name: value, object: [] })
    },

    getTranslate(items){
      return items
    },

    getSidebarMenu(){

      let sidebar = this.$store.getters['projectSidebar/getSidebar']

      this.projectExtra = Boolean(localStorage.getItem('EXTRA_PROJECT_' + this.project.slug))
      this.statusProjectExtra() 

      if ( sidebar !== null ) {
        this.menu = sidebar
        return 
      }

      Axios()
        .get('/projects/sidebar/' +
              '?company_slug=' +
              this.$route.params.companySlug+
              '&project_slug=' +
              this.$route.params.projectSlug
        )
        .then((response) => {

          this.menu = this.getTranslate(response.data.data)
          this.$store.dispatch('projectSidebar/setSidebar', this.menu)

        })
        .catch((e) => {
          console.error(e)
        })
    },

    getProject() {

      let project = this.$store.getters['project/getProject']

      if (  project !== null && 
            project.slug === this.$route.params.projectSlug && 
            project.company.slug === this.$route.params.companySlug ) 
      {
        this.project = project
        this.getSidebarMenu()
        return 
      }

      Axios()
        .get('projects/' + this.$route.params.projectSlug + '/?company_slug=' + this.$route.params.companySlug)
        .then((response) => {
          if (response.data.message) {
            this.$router.push({
              name: 'workspaces.projects',
              query: {
                project_access: 'denied',
              },
            })
          }

          this.project = response.data.data
          this.$store.dispatch('projectSidebar/setSidebar', null)
          this.$store.dispatch('project/setProject', this.project)

          this.getSidebarMenu()
          this.setBackground()
          
          
        })
        .catch((e) => {
          console.error(e)
        })
    },

    setBackground(){
      
      if ( document.getElementById('page-content') !== null ){
        if (this.background && this.project.background) {
          document.getElementById('page-content').style.backgroundImage = 'url(' + this.project.background + ')'
        } else {
          /*
          document.getElementById('page-content').style.backgroundImage =
            'url(//gitscrum-static.s3.amazonaws.com/img/pattern-simple.png)'
          document.getElementById('page-content').classList.add('background-no-cover')
          */
        }
      }

    },

    onToggleCollapse(collapsed) {

      if(collapsed){
        localStorage.setItem('MENU_PROJECT_COLLAPSED', 1)

        if (document.getElementById('page-padding-left'))
          document.getElementById('page-padding-left').style.paddingLeft = "50px";
       
      } else {

        localStorage.removeItem('MENU_PROJECT_COLLAPSED')
        
        if (document.getElementById('page-padding-left'))
          document.getElementById('page-padding-left').style.paddingLeft = '240px';
        
      }

    },
    getMenuProjectState(){
      if (isMobile){
        return true
      }
      let state = Boolean(localStorage.getItem('MENU_PROJECT_COLLAPSED'))
      return state
    },

    openTaskFilters(){
      this.sidebarStatus = true
      document.getElementById('page-content').style.paddingLeft = "260px";
    },

    closeTaskFilters(){
      this.sidebarStatus = false
      document.getElementById('page-content').style.paddingLeft = "10px";
    },

    statusProjectExtra(){

      if (this.projectExtra){
        this.projectExtraIcon = 'minus-square'
        localStorage.setItem('EXTRA_PROJECT_' + this.project.slug, true)
      } else {
        this.projectExtraIcon = 'plus-square'
        localStorage.removeItem('EXTRA_PROJECT_' + this.project.slug)
      }

      this.projectExtra = !this.projectExtra 

    }
  }
}
</script>

<template>
  <div>
    <b-sidebar
      v-if="!isMobile"
      id="sidebar-task-filter"
      title="Advanced Filters"
      @shown="openTaskFilters"
      @hidden="closeTaskFilters">
      <ProjectBoardFilters
        v-if="authorize('header', 'filters') && background"
        :sidebar-status="sidebarStatus"></ProjectBoardFilters>
    </b-sidebar>
    <b-progress class="project-progress" :value="project.percent" :max="100"></b-progress>
    <SidebarMenu :menu="menu" :collapsed="getMenuProjectState()" @toggle-collapse="onToggleCollapse">
      <div slot="header">
        <div id="sidebar-left" class="sidebar-left">
          <div class="d-flex justify-content-start">
            <router-link
              :to="{
                name: 'projects.board',
                params: {
                  companySlug: this.$route.params.companySlug,
                  projectSlug: this.$route.params.projectSlug,
                },
              }" class="header-project-title" >
              <ProjectVisibility :visibility="project.visibility"></ProjectVisibility>
              <span v-text="project.pure_name"></span>
            </router-link>
          </div>
          <hr>
          <div class="statusProjectExtra">
            <font-awesome-icon 
              :icon="['far', projectExtraIcon]" 
              class="cursor-pointer" 
              style="font-size:18px; color: #909CB8;" 
              @click="statusProjectExtra" />
          </div>
          <div v-show="projectExtra">
            <div class="header-project-logo">
              <img :src="project.logo" :alt="project.name" />
              <div>
                <p class="vsm--header">{{ $t('Project Leader') }}</p>
                <router-link
                  :to="{
                    name: 'profile.user',
                    params: { username: project.owner.username },
                  }">
                  {{project.owner.name}}
                </router-link>
              </div>
            </div>
            <div v-if="authorize('header', 'users')" id="project-team-members" class="header-project-team-members">
              <h3>{{ $t('Project Team Members') }}</h3>
              <ListUsers
                :users="project.users"
                :link="true"
                :limit="50"
                :wrap="true"
                size="22"
                class="d-none d-sm-block"></ListUsers>
            </div>
            <div class="header-project-collaboration">
              <b-progress class="project-list-progress" :value="project.my_contribution" :max="100"></b-progress>
              <p>{{ $t('My contribution') }}: {{ parseFloat(project.my_contribution) | percent(0) }}</p>
            </div>
          </div>
        </div>
      </div>
      <div slot="footer" class="sidebar-left">
        <div class="vsm--list">
          <div v-if="authorize('settings', 'read')" class="vsm--item">
            <router-link
              :to="{
                name: 'project.settings.details',
                params: {
                  companySlug: this.$route.params.companySlug,
                  projectSlug: this.$route.params.projectSlug,
                } }" class="router-link-active vsm--link vsm--link_level-1">
              <span class="icon-size mr-10-px"><font-awesome-icon :icon="['far', 'cog']"/></span>
              <span class="vsm--title">{{ $t('Project Settings') }}</span>
            </router-link>
          </div>
          <ProjectMenuSettings
            v-if="authorize('settings', 'read')"
            class="d-none d-sm-block" ></ProjectMenuSettings>
        </div>
      </div>
    </SidebarMenu>
    <ProjectIntegrations></ProjectIntegrations>
  </div>
</template>
