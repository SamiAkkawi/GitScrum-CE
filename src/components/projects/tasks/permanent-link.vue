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
  <b-form-group
    class="task-permanent-link badge badge-light wd-100 text-left">
    <b-link @click="statusPermanentOpen">
      <font-awesome-icon 
        :icon="['far', projectExtraIcon]" 
        class="cursor-pointer" 
        style="font-size:18px; color: #909CB8;" 
         />
      {{ $t('Permanent Link') }}
      
    </b-link>
    <b-input-group v-show="projectExtra">
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
  </b-form-group>
</template>