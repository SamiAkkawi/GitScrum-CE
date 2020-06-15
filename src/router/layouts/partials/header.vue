<script>
import Axios from '@utils/axios'
import MasterHeaderCreate from '@layouts/partials/header-menu-create'
import HeaderLanguage from '@layouts/partials/header-language'
import Logo from '@layouts/partials/logo'
// import HeaderNotifications from '@layouts/partials/header-notifications'
import HeaderUser from '@layouts/partials/header-user'
import HeaderGift from '@layouts/partials/header-gift'
import HeaderChangeCompany from '@layouts/partials/header-change-company'
import GlobalSideBar from '@layouts/partials/global-side-bar'
import VOffline from 'v-offline';
import { isMobile } from 'mobile-device-detect';
import MyNextTasks from '@components/utils/my-next-tasks'

export default {
  components: {
    MasterHeaderCreate,
    HeaderLanguage,
    Logo,
    // HeaderNotifications,
    HeaderUser,
    HeaderGift,
    HeaderChangeCompany,
    GlobalSideBar,
    MyNextTasks,
    VOffline
  },
  props: {
    hidden: {
      type: Boolean,
      required: false,
      default: false,
    },
    sharing: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      gsNight: false,
      isMobile: isMobile,
      loading: false,
      details: null,
      onLine: null,
      onlineSlot: 'online',
      offlineSlot: 'offline', 
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      currentUser: JSON.parse(localStorage.getItem('CURRENT_USER')),
      statusMyNextTask: false,
    }
  },
  mounted() {
    if (this.hidden) {
      this.getInfosDetails()
    }
    this.gsNight = !localStorage.getItem('GS_NIGHT')
    this.changeTheme()
    this.gsNight = !this.gsNight
  },
  methods: {
    amIOnline(e) {
      this.onLine = e;
    },
    getInfosDetails() {
      this.loading = true
      Axios()
        .get('share-board/public/project/' + this.$route.params.token + '/')
        .then((response) => {
          this.details = response.data.data

          if ( this.sharing ){
            document.title = this.details.project + ' ' + this.$t('project shared board')
            document.description = this.$t("Share-text")
          }

          if (this.details.background) {
            document.getElementById('page-content').style.backgroundImage = 'url(' + this.details.background + ')'
          } else {
            document.getElementById('page-content').style.backgroundImage =
              'url(//gitscrum-static.s3.amazonaws.com/img/pattern-simple.png)'
            document.getElementById('page-content').classList.add('background-no-cover')
          }
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },

    goto(route){
      this.$ga.event('header_bar', 'click_link', route.name, 1)
      this.$router.push(route)
    },

    getCompleteURL(){
      return window.location;
    },

    openSidebar(){
      if (this.statusMyNextTask){
        this.statusMyNextTask = false
      } else {
        this.statusMyNextTask = true;
      }
    },
    changeTheme(){
      const el = document.body;

      if ( this.gsNight ){
        el.classList.remove('GitScrum-Night');
        localStorage.removeItem('GS_NIGHT')
      } else {
        el.classList.add('GitScrum-Night');
        localStorage.setItem('GS_NIGHT', true)
      }
    },
    checkUpgrade() {
      if (
        ( this.currentCompany.subscription !== null ||
        this.currentCompany.subscription === 'free' ) && 
        this.currentCompany.owner.username === this.currentUser.username ) {
        return true
      }

      return false
    },
  },
}
</script>

<template>
  <div id="header-content">
    <VOffline
      online-class="online"
      offline-class="offline"
      @detected-condition="amIOnline">
      <template v-slot:[onlineSlot] :slot-name="onlineSlot">
        <font-awesome-icon :icon="['fal', 'wifi-slash']" />
        &nbsp;
        {{ $t('You’re offline. Check your connection.')}}
      </template>
    </VOffline>
    <b-navbar id="header-navbar" toggleable="md" tag="ul">
      <Logo></Logo>
      <HeaderUser v-if="isMobile"></HeaderUser>
      <b-navbar-toggle v-if="!sharing" target="nav-collapse"></b-navbar-toggle>
      <b-collapse v-if="!sharing" id="nav-collapse" is-nav>
        <b-navbar-nav>
          <HeaderLanguage></HeaderLanguage>
          <li>
            <div class="dark-mode">
              <b-form-checkbox 
              v-model="gsNight" 
              switch 
              size="sm" 
              @change="changeTheme">{{ $t('Dark') }}</b-form-checkbox>
            </div>
          </li>
          <li v-if="checkUpgrade()">
            <a href="https://site.gitscrum.com/pricing-growing-companies" target="_blank">
              <b-badge class="badge-upgrade">{{ $t('Upgrade to Business Unlimited - Only $49') }}</b-badge>
            </a>
          </li>
          <li v-if="!isMobile">
            <b-button 
              v-shortkey="['ctrl', 'n']" 
              v-b-tooltip.hover 
              :title="$t('Shortcut : CTRL + N')"
              class="btn btn-secondary" 
              @shortkey="openSidebar" 
              @click="openSidebar">
              <span>{{ $t('My Next Tasks') }}</span></b-button>
          </li>
          <li>
            <b-button 
              v-shortkey="['ctrl', 'p']" 
              v-b-tooltip.hover 
              :title="$t('Shortcut : CTRL + P')"
              class="btn btn-secondary" 
              @shortkey="goto({ name: 'workspaces.projects' })" 
              @click="goto({ name: 'workspaces.projects' })">
              <span>{{ $tc('Workspaces', 2) }}</span></b-button>
          </li>          
          <li v-if="!isMobile">
            <b-button 
            v-if="currentCompany.owner.username === currentUser.username"
            v-shortkey="['ctrl', 'm']" 
            v-b-tooltip.hover 
            :title="$t('Shortcut : CTRL + M')"
            class="btn btn-secondary" 
            @shortkey="goto({ name: 'marketplace.templates' })" 
            @click="goto({ name: 'marketplace.templates' })">
            <span>{{ $t('Marketplace') }}</span></b-button>
          </li>
        </b-navbar-nav>
        <b-navbar-nav class="ml-auto">
          <HeaderChangeCompany></HeaderChangeCompany>
          <MasterHeaderCreate></MasterHeaderCreate>
          <!--<HeaderNotifications></HeaderNotifications>-->
          <HeaderGift v-if="!isMobile"></HeaderGift>
          <HeaderUser v-if="!isMobile"></HeaderUser>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <slot name="submenu"> </slot>
    <b-sidebar 
      id="my-next-tasks"
      v-model="statusMyNextTask" 
      no-header
      backdrop
      shadow
      left
      class="my-next-tasks">
       <MyNextTasks></MyNextTasks>      
    </b-sidebar>
    <div v-if="!currentUser.confirmed" class="footer">
      {{ $t('Look for the verification email in your inbox and click the link in that email.') }}
    </div>
  </div>
</template>