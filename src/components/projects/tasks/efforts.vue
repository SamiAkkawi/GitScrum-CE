<script>
import { taskManager } from '@state/helpers'

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
      effort: []
    }
  },
  computed: {
    ...taskManager
  },
  watch: {
    statusTask(data){
      if ( data.item.name === 'effort.change' && this.task.uuid === data.item.uuid ){
        this.effort = data.item.object
      }
    },
  },
  created(){
    this.setType()
  },
  methods: {
    setType(){
      this.effort = this.task.effort
    }
  }
}
</script>

<template>
  <span 
    v-if="effort !== null" 
    class="badge badge-light mg-r-5">
    {{ effort.title }} - {{ effort.effort }} {{ $t('points') }}
  </span>
</template>
