<script>
import Layout from '@layouts/tpl-main'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import SideBar from '@components/companies/side-bar'
import ListUsers from '@components/utils/list-users'
import Alert from '@components/utils/alert'
import Ads from '@components/utils/ads'
import ButtonLoading from '@components/utils/button-loading'

export default {
  page: {
    title: 'Company Team',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, TitleLoading, Alert, SideBar, ListUsers, Ads, ButtonLoading },
  data() {
    return {
      teammates: [],
      inviteNames: [],
      inviteEmails: [],
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      activeUsers: 0,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      alertMessage: '',
      alertStatus: false,
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
      loading: false,
      
      showInvite: false,
      inviteCompanyLink: '',
      invitationLinkLoading: false,
      enableShareableLink: false,
      linkCopied: false,
      emailExpression: /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,24}))$/,
      sendInvitesLoading: false,

      totalInvites: [0,1,2],

      fields: [
        {
          key: 'user',
          label: 'Team Member',
        },
        {
          key: 'status',
          label: 'Status',
        },
        {
          key: 'create_project',
          label: 'Can create project',
        },
      ],
    }
  },
  mounted() {
    this.getTeammates(this.currentPage)
  },
  methods: {
    scrollToTop() {
      window.scroll({
        top: 0,
        left: 0,
        behavior: 'smooth',
      })
    },

    getTeammates(page) {
      this.scrollToTop()
      this.loading = true
      this.activeUsers = 0
      Axios()
        .get('team-members/?company_slug=' + this.currentCompany.slug + '&page=' + page)
        .then((response) => {
          if (response.data.data.length === 0 && page !== 1) {
            this.getTeammates(1)
            return
          }
          this.teammates = response.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page

          this.getActiveUsers()

          this.loading = false
        })
        .catch((e) => {
          this.loading = false
          // console.error(e)
        })
    },

    getActiveUsers() {
      Axios()
        .get('team-members/active-users/?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.activeUsers = response.data.data
        })
    },

    updateStatusCurrent(username) {
      Axios()
        .put(
          'team-members/toggle/ArchiveOrUnarchive/?company_slug=' + this.currentCompany.slug + '&username=' + username
        )
        .then((_) => {
          this.getTeammates(this.currentPage)
        })
      // .catch((e) => {
      //   console.error(e)
      // })
    },

    remove(username, index) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete('team-members/?company_slug=' + this.currentCompany.slug + '&username=' + username, {})
          .then((response) => {
            this.alertMessage = this.$t('Team member was deleted successfully')
            this.alertStatus = true
            this.getTeammates(this.currentPage)
          })
        }
      })
      
    },

    canCreateProject(username, item) {
      Axios()
        .put('team-members/toggle/CreateProject/?company_slug=' + this.currentCompany.slug + '&username=' + username)
        .then((response) => {
          if ( !item.status.can_create_projects) {
item.status.can_create_projects = true;
          } else {
            item.status.can_create_projects = false
          }
        })
      // .catch((e) => {
      //   console.error(e)
      // })
    },

    toggleShowInvite() {
      this.showInvite = !this.showInvite
      if (this.showInvite) {
        this.alertStatus = false
        if (!this.inviteCompanyLink) {
          this.getInvitationLink()
        }
      }
    },

    getInvitationLink() {
      this.invitationLinkLoading = true

      Axios()
        .get('team-members/users/invite/shareable?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.enableShareableLink = response.data.data.share_enabled
          if (this.enableShareableLink) {
            this.inviteCompanyLink = response.data.data.url
          }
          this.invitationLinkLoading = false
        })
        .catch((e) => {
          this.invitationLinkLoading = false
        })
    },

    handleShareableLink(status) {
      this.invitationLinkLoading = true
      this.linkCopied = false

      Axios()
        .post('team-members/users/invite/shareable?company_slug=' + this.currentCompany.slug + '&status=' + status)
        .then((response) => {
          this.enableShareableLink = response.data.data.share_enabled
          this.inviteCompanyLink = response.data.data.url
          this.invitationLinkLoading = false
        })
        .catch((e) => {
          this.invitationLinkLoading = false
        })
    },

    updateInvitationLink() {
      this.invitationLinkLoading = true
      this.linkCopied = false
      this.inviteCompanyLink = ''

      Axios()
        .put('team-members/users/invite/shareable?company_slug=' + this.currentCompany.slug)
        .then((response) => {
          this.inviteCompanyLink = response.data.data.url
          this.invitationLinkLoading = false
        })
        .catch((e) => {
          this.invitationLinkLoading = false
        })
    },

    onCopy() {

      this.$copyText(this.tokenShareableBoard)
      this.linkCopied = false
      
    },

    resetInviteFields() {
      this.inviteNames = []
      this.inviteEmails = []
    },

    checkForm() {
      this.errors = []

      this.inviteEmails.forEach((email) => {
        if (!this.validEmail(email)) {
          this.errors.push(email)
        }
      })

      if (this.errors.length) {
        return false
      }

      return true
    },

    validEmail(email) {
      return this.emailExpression.test(email)
    },

    sendInvites() {
      this.alertStatus = false
      this.alertMessage = ''
      if (this.inviteEmails.length === 0) {
        this.alertMessage = this.$t('Email is mandatory')
        this.alertStatus = true
      } else {
        if (!this.checkForm()) {
          this.alertMessage = this.$t('One or more email field is incorrect')
          this.alertStatus = true
          return
        }
        this.sendInvitesLoading = true

        Axios()
          .post('team-members/invite/?company_slug=' + this.currentCompany.slug, {
            emails: this.inviteEmails,
            names: this.inviteNames,
          })
          .then((response) => {
            this.alertMessage = this.$t('Invitations successfully sent')
            this.alertStatus = true
            this.sendInvitesLoading = false
            this.resetInviteFields()
          })
      }
    },
  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
      <TitleLoading
        :title="$t('Company Team Members')"
        :loading="loading"
      ></TitleLoading>
    </template>

    <div slot="content" class="company-team pt-10px">

      <div class="row mb-30-px">
        <div class="col-md-2">
          <SideBar></SideBar>
        </div>
        <div class="col-md-9 offset-md-1">

          <div class="card">
            <div class="card-body">
              <div class="d-flex justify-content-between">
                <div>
                  {{ currentCompany.stats.plan.title }} - 
                  <span v-if="currentCompany.stats.plan.users == 'Unlimited'">
                    {{ $t('Unlimited users') }}
                  </span>
                  <span v-else>
                    {{ $t('up to users', { users: currentCompany.stats.plan.users }) }}
                  </span>
                  <span v-if="!activeUsers.invitation && !loading" class="d-block tx-12-px font-weight-bold txt-EF5958">
                    {{ $t('You need to upgrade your account') }}
                  </span>
                </div>

                <div v-not-visible="'tablet'">
                  <a v-if="activeUsers.invitation" class="small fw-600" href="javascript:;" @click="toggleShowInvite">
                    {{ $t('Invite Members') }}
                    <font-awesome-icon :icon="['far', 'angle-down']" style="position:relative; margin-left:4px;" />
                  </a>
                  <router-link :to="{ 'name': 'companies.teams.log' }" class="ml-3 small">
                    {{ $t('See the people you have already invited') }}  
                    <font-awesome-icon :icon="['far', 'angle-right']" class="ml-1" />
                  </router-link>
                </div>
              </div>
            </div>
          </div>

          <div v-show="!loading && activeUsers">
            <Ads v-if="currentCompany.subscription === 'free' || !activeUsers.invitation" type="large"></Ads>
          </div>

          <Alert :message="alertMessage" :status="alertStatus" class="mt-2"></Alert>

          <div v-show="showInvite" v-not-visible="'tablet'" class="card">
            <div class="card-body">

              <div class="invite-container">
                <div class="d-flex justify-content-between">
                  <b-form-checkbox
                    v-model="enableShareableLink"
                    :disabled="invitationLinkLoading"
                    @change="handleShareableLink"
                    class="mb-3">
                    <TitleLoading
                      :title="$t('Shareable Link')"
                      :subtitle="$t('Anyone with this link can join the company')"
                      :loading="invitationLinkLoading">
                    </TitleLoading>
                  </b-form-checkbox>

                  <a v-show="enableShareableLink"
                    href="javascript:;"
                    class="txt-68748F fw-500 lh-15-px tx-10-px tx-uppercase mt-5-px"
                    @click="updateInvitationLink" >
                    {{ $t('Update Link') }}
                  </a>
                </div>

                <b-input-group v-show="enableShareableLink" class="mt-2">
                  <b-input v-model="inviteCompanyLink" 
                    :readonly="true" 
                    autocomplete="off"></b-input>
                  <b-input-group-append>
                    <b-button 
                    v-clipboard:copy="inviteCompanyLink"
                    v-clipboard:success="onCopy"
                    variant="outline-secondary" >
                      <font-awesome-icon :icon="['far', 'copy']" />
                    </b-button>
                  </b-input-group-append>
                </b-input-group>
                  
                <div class="d-flex justify-content-between mt-3 mb-2">
                  <TitleLoading
                    :title="$t('Invite your Team')"
                    :subtitle="$t('Bring teammates to your company. After accepting the invitation, you must add teammates to the projects.')"
                    :loading="sendInvitesLoading"
                  >
                  </TitleLoading>
                </div>

                <div class="invite-form invite-form-company">
                  <div v-for="invite in totalInvites" :key="invite" class="form-row">
                    <div class="col-md-6 form-group">
                      <div role="group">
                        <label for="input-live">{{ $t('Full Name') }}</label>
                        <b-form-input
                          v-model="inviteNames[invite]"
                          type="text"
                          :placeholder="$t('Full Name')"
                          trim
                        ></b-form-input>
                      </div>
                    </div>
                    <div class="col-md-6 form-group">
                      <div role="group">
                        <label for="input-live">{{ $t('Email') }}</label>
                        <b-form-input
                          v-model="inviteEmails[invite]"
                          type="email"
                          :placeholder="$t('Email')"
                          trim
                        ></b-form-input>

                      </div>
                    </div>
                  </div>

                  <div class="mt-3 mb-2">
                    <hr>
                    <div class="d-flex justify-content-between">
                      <div></div>
                      <ButtonLoading
                      :loading="loading"
                      :title="$t('Send Invite')"
                      :title-loading="$t('Sending')"
                      type="btn-md"
                      @action="sendInvites"></ButtonLoading>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <b-table class="table-team-members" striped hover="" :items="teammates" :fields="fields" >
            <template v-slot:cell(user)="data">
              <div class="box-useravatar">
                <ListUsers :user="data.item.user" :link="true" size="22"></ListUsers>
                <router-link
                :to="{
                  name: 'profile.user',
                  params: { username: data.item.user.username },
                }" >
                  {{data.item.user.name}}
                </router-link>
              </div>
            </template>
            <template v-slot:cell(status)="data">
              <div v-if="data.item.status.current === 'active'" >
                <div class="d-flex align-items-center">
                  <span class="badge" style="background:#26DC8E;margin-top:2px;">
                    {{ $t('Active') }}
                     <font-awesome-icon 
                      v-if="!data.item.status.company_owner"
                      v-b-popover.hover.top="$t('Accepted at XXX', 
                      { datetime: data.item.accepted_at })" 
                      :icon="['far', 'calendar-check']" 
                      class="ml-5-px" />
                  </span>
                  <b-link 
                    v-if="!data.item.status.company_owner"
                    href="javascript:;" 
                    class="small" 
                    @click="updateStatusCurrent(data.item.user.username)">{{ $t('Deactivate') }}</b-link>
                </div>
              </div>
              <div v-else class="d-flex align-items-center">
                <span class="badge" style="background:#A7AFB7;margin-top:2px;">{{ $t('Disabled') }}
                  <font-awesome-icon 
                    v-b-popover.hover.top="$t('XXX does not have access to your company and projects', 
                      { username: data.item.user.name })" 
                    :icon="['fal', 'question-circle']" 
                    class="ml-5-px" />
                </span>
                <b-link 
                    v-if="!data.item.status.company_owner"
                    href="javascript:;" 
                    class="small" 
                    @click="updateStatusCurrent(data.item.user.username)">{{ $t('Activate') }}</b-link>
              </div>
            </template>
            <template v-slot:cell(create_project)="data">
              <div class="d-flex justify-content-between">
                <div>
                  <div v-if="!data.item.status.company_owner" class="d-flex align-items-center">
                    <b-form-checkbox
                      switch
                      :checked="data.item.status.can_create_projects"
                      @change="canCreateProject(data.item.user.username, data.item)">
                    </b-form-checkbox>
                    <font-awesome-icon 
                      v-show="data.item.status.can_create_projects" 
                      v-b-popover.hover.top="$t('XXX can create projects within his company', 
                        { username: data.item.user.name })" 
                      :icon="['fal', 'question-circle']" 
                      class="ml-5-px" />
                    <font-awesome-icon 
                      v-show="!data.item.status.can_create_projects" 
                      v-b-popover.hover.top="$t('XXX cannot create projects within his company', 
                        { username: data.item.user.name })" 
                      :icon="['fal', 'question-circle']" 
                      class="ml-5-px" />
                  </div>
                  <div v-else>
                    <strong  class="txt-68748F tx-12-px">Company Owner</strong>
                  </div>
                </div>
                <div>
                  <b-link v-if="!data.item.status.company_owner" 
                    @click="remove(data.item.user.username, data.index)">
                    <font-awesome-icon 
                      :icon="['far', 'trash-alt']" />
                  </b-link>
                </div>
              </div>
              
              
            </template>
          </b-table>

          <div v-if="totalPages > 1" class="d-flex justify-content-center mt-4">
            <b-pagination
              v-model="currentPage"
              hide-goto-end-buttons
              class="paginator"
              :total-rows="totalRows"
              :per-page="perPage"
              @change="getTeammates"
            >
              <template slot="prev-text">
                <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
              </template>
              <template slot="next-text">
                <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
              </template>
            </b-pagination>
          </div>

        </div>
      </div>
    </div>
  </Layout>
</template>
