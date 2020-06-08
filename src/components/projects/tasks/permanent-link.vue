<script>
export default {
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
      taskUrl: '',
      projectExtraIcon: 'plus-square',
      projectExtra: false
    }
  },
  created(){
    this.formatLink()
  },
  methods: {
    formatLink(){
      let url = this.$router.resolve({
          name: 'projects.board.task-details',
          params: {
            companySlug: this.task.company.slug,
            projectSlug: this.task.project.slug,
            taskSlug: this.task.uuid,
          },
        }).href

        this.taskUrl = this.getDomain( url.substr(1)) 
    },
    onCopy(){
      this.$copyText(this.taskUrl)
    },
    statusPermanentOpen(){
      if (this.projectExtra){
        this.projectExtraIcon = 'plus-square'
        this.projectExtra = false
      } else {
        this.projectExtraIcon = 'minus-square'
        this.projectExtra = true
      }

    }
  }
}
</script>

<template>
<div class="task-permanent-link">
  <b-button 
  v-if="authorize('tasks', 'create')" 
  v-b-toggle.permanent-link-icon 
  class="btn btn-secondary btn-block"
  :style="(task.type) ? 'color: ' + 
  invertColor(task.type.color, true) + 
  ';background: ' + 
  opacityColor(task.type.color, '0.6') : ''"
  v-text="$t('Permanent Link')"></b-button>
  <b-collapse id="permanent-link-icon">
    <b-card>
      <b-input-group>
        <b-input v-model="taskUrl" 
          :readonly="true" 
          autocomplete="off"></b-input>
        <b-input-group-append>
          <b-button 
          v-clipboard:copy="taskUrl"
          v-clipboard:success="onCopy"
          variant="outline-secondary" >
            <font-awesome-icon :icon="['far', 'copy']" />
          </b-button>
        </b-input-group-append>
      </b-input-group>
    </b-card>
  </b-collapse>

  
</div>
</template>