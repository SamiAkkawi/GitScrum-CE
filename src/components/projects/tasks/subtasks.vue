<script>
import Axios from '@utils/axios'
import ListTasks from '@components/projects/tasks/list-tasks'
import { taskManager } from '@state/helpers'

export default {
  components: { ListTasks },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
    modalFlag: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      loading: true,
      subtasks: [],
      visible: true,
    }
  },
  computed: {
    ...taskManager,
  },
  watch: {
    statusTask(data){
      if ( data.item.name === 'subtask.addList' ){
        this.subtasks.push(data.item.object)
      }
    },
  },
  created() {
    this.getSubtasks()
  },
  methods: {
    getSubtasks() {
      Axios()
        .get(
          'tasks/' +
            this.task.uuid +
            '/sub-tasks/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug
        )
        .then((response) => {
          this.subtasks = response.data.data
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },
  },
}
</script>

<template>
  <b-row v-if="subtasks[0]" class="mb-3">
    <b-col cols="1" class="task-left-icon">
      <font-awesome-icon :icon="['far', 'tasks']" />
    </b-col>
    <b-col class="task-left-content">
      <h5 class="mb-3">{{ $t('Subtasks') }}</h5>
      <ListTasks v-show="!loading" :items="subtasks" :modal-flag="modalFlag"></ListTasks>
    </b-col>
  </b-row>
</template>
