<script>
import Axios from '@utils/axios'
import ListComments from '@components/utils/list-comments'
import CommentEditor from '@components/utils/comment-editor'


export default {
  components: {
    ListComments,
    CommentEditor,
  },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
  },
  data() {
    return {
      comments: [],
      loading: true,
      btnLoading: false,
      visible: true,
      txtComment: '',
      selected: 'DESC',
      options: [
        { value: 'ASC', text: 'Ascending' },
        { value: 'DESC', text: 'Descending' },
      ]
    }
  },
  created() {
    this.getComments()
  },
  methods: {
    handleSaveComment(comment) {
      this.txtComment = ''
      this.getComments()
    },
    getComments() {
      this.loading = true
      Axios()
        .get(
          'comments/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&commentable_type=issues' +
            '&commentable_id=' +
            this.task.uuid + 
            '&sort=' + this.selected
        )
        .then((response) => {
          this.comments = response.data.data
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },
    updateText(){
      this.getComments()
    },
  },
}
</script>

<template>
      <b-row class="mb-3">
        <b-col cols="1" class="task-left-icon">
          <font-awesome-icon :icon="['far', 'comments-alt']" />
        </b-col>
        <b-col class="task-left-content">
          <div class="d-flex justify-content-between">
            <h5>
              {{ $t('Comments') }}
            </h5>
            <div v-if="authorize('tasks', 'read')" v-show="comments[0]" class="d-flex justify-content-end" style="position:relative; top:-6px;">
              <b-spinner v-show="loading" :label="$t('Loading')" small style="width: 18px;height: 16px;border-width: 2px; position:relative; top:7px; right:8px;"></b-spinner>
              <b-form-select v-model="selected" :options="options" size="sm" @change="getComments" ></b-form-select>
            </div>
          </div>
          <CommentEditor
            v-if="authorize('comments', 'read')"
            :edit-mode="false"
            :cancel-mode="false"
            :company-slug="task.company.slug"
            :project-slug="task.project.slug"
            :commentable-id="task.uuid"
            :btn-title="$t('Send a comment')"
            :btn-subtitle="$t('Sending')"
            :redirect="false"
            commentable-type="issues"
            @text="updateText"></CommentEditor>
          <hr>
          <ListComments 
            :data="comments" 
            :company-slug="task.company.slug" 
            :project-slug="task.project.slug" 
            :main-comment-id="task.uuid" 
            @updateComments="getComments">
          </ListComments>
        </b-col>
      </b-row>
</template>
