<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import TitleLoading from '@components/utils/title-loading'
import Mentions from '@components/utils/mentions'
import CommentEditor from '@components/utils/comment-editor'
import ListOtherDiscussions from '@components/projects/discussions/list-other-discussions'
import ListComments from '@components/utils/list-comments'
import ListUsers from '@components/utils/list-users'

export default {
  page: {
    title: 'Discussions',
    meta: [{ name: '', content: '' }],
  },
  components: {
    Layout,
    TitleLoading,
    Mentions,
    CommentEditor,
    ListOtherDiscussions,
    ListComments,
    ListUsers
  },
  data() {
    return {
      loading: true,
      replyText: '',
      editComment: false,
      stDisabled: true,
      newDiscussion: '',
      txtComment: '',
      item: [],
      createDiscussionOpen: false,
      
      user: JSON.parse(localStorage.getItem('CURRENT_USER')),
    }
  },
  mounted() {
    this.getDiscussions()
  },
  methods: {
    getDiscussions() {
      Axios()
        .get(
          'comments/' +
            this.$route.params.discussionId +
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&commentable_type=projects' +
            '&commentable_id=null'
        )
        .then((response) => {
          this.item = response.data.data
          this.loading = false
        })
        .catch((error) => {
          console.error(error)
        })
    },

    handleSaveComment(comment) {
      this.txtComment = ''
      this.item.replies.push(comment)
    },

    updateText(text){
      this.item.comment_mention = text
      this.editComment = false
    },

    reply(text){
      this.getDiscussions()

    },

    cancel(){
      this.editComment = false
    },

    deleteComment(){

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){

          Axios()
          .delete(
            'comments/' +
              this.$route.params.discussionId +
              '/?company_slug=' +
              this.$route.params.companySlug +
              '&project_slug=' +
              this.$route.params.projectSlug
          )
          .then((response) => {
            this.$router.push({
              name: 'projects.discussions',
              params: { companySlug: this.$route.params.companySlug , projectSlug: this.$route.params.projectSlug },
            })
          })
        }
      })

    }

  },
}
</script>

<template>
  <Layout>

    <template slot="header-left">
     <TitleLoading
        :title="$t('Discussions')"
        :subtitle="$t('Discussion is essential to promote dialogue for project’s goals')"
        :loading="loading"
      >
      </TitleLoading>
    </template>

    <div slot="content" class="project-discussions pt-15px">

      <b-container>
        <b-row>
          <b-col cols="8">
            <b-card>
              <template v-slot:header>
                <div class="d-flex align-content-center justify-content-between">
                  <h6 class="font-weight-bold mb-0">{{$t('Discussion')}}</h6>
                  <div v-if="authorize('discussions', 'create')" class="d-flex justify-content-end">
                    <b-link class="mr-15-px" @click="deleteComment">{{ $t('Delete') }}</b-link>
                    <b-link @click="editComment = true">{{ $t('Edit Discussion') }}</b-link>
                  </div>
                </div>
              </template>

              


              <h2 v-show="!editComment" class="h6" v-html="item.comment_mention"></h2>
              <div v-show="editComment">
              <CommentEditor
              :edit-mode="true"
              :cancel-mode="true"
              :edit-content="item.comment_mention"
              :comment-id="item.id"
              :company-slug="$route.params.companySlug"
              :project-slug="$route.params.projectSlug"
              @text="updateText"
              @cancel="cancel" ></CommentEditor>
              </div>
              <div class="d-flex justify-content-between">
                <div class="box-useravatar mr-2 pr-2">
                  <ListUsers :user="item.user" :link="true" size="14"></ListUsers>
                  <router-link
                  :to="{
                    name: 'profile.user',
                    params: { username: item.user.username } }" >
                    {{item.user.name}}
                  </router-link>
                </div>
                <div style="font-size: 12px;color: #55617c;font-weight: 500;">
                    {{ $t('Posted at') }}
                  <span v-text="item.created_at.date_for_humans"></span>
                </div>
              </div>
            </b-card>

            <b-card>
              <CommentEditor
              :btn-title="$t('Topic reply')"
              :parent-id="item.id"
              :cancel-mode="false"
              :edit-mode="false"
              :comment-id="item.id"
              :company-slug="$route.params.companySlug"
              :project-slug="$route.params.projectSlug"
              @text="reply"
              @cancel="cancel"></CommentEditor>
            </b-card>

            <b-card>
              <ListComments
              :data="item.replies"
              :show-title="true"
              :company-slug="$route.params.companySlug"
              :project-slug="$route.params.projectSlug">
              </ListComments>
            </b-card>
          </b-col>
          <b-col cols="3" offset="1">
            <b-card :header="$t('Other discussions')">
              <ListOtherDiscussions 
              :company-slug="$route.params.companySlug" 
              :project-slug="$route.params.projectSlug">
              </ListOtherDiscussions>
            </b-card>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </Layout>
</template>

