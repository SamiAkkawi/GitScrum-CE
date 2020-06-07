<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'
import { taskManager } from '@state/helpers'

export default {
  components: { ButtonLoading },
  props: {
    task: {
      type: Object,
      required: true
    },
    dropdown: {
      type: Boolean,
      required: false,
      default: false
    },
  },
  data() {
    return {
      types: [],
      currentPage: 0,
      totalPages: 1,
      searchTerm: '',
      searchLoading: false,
    }
  },
  /*
  computed: {
    ...taskManager
  },
  */
  watch: {
    statusTask(data){
      if ( data.item.name === 'type.change' && this.task.uuid === data.item.uuid ){
        alert('')
        this.task.type = data.item.object
      }
    },
  },
  methods: {
    ...taskManager,

    loadMore() {
      this.currentPage = this.currentPage + 1
      if (this.currentPage <= this.totalPages) {
        this.loading = true
        this.getTaskTypes()
      }
    },

    getTaskTypes() {
      Axios()
        .get(
          'project-templates/type/?company_slug=' + 
          this.task.company.slug + 
          '&project_slug=' + 
          this.task.project.slug +
          '&search=' + 
          this.searchTerm +
          '&page=' +
          this.currentPage
        )
        .then((response) => {
          
          this.types = response.data.data
          this.searchLoading = false
          this.loading = false
        })
    },
    changeTaskType(type) {

      this.actionTask({ name: 'type.change', uuid: this.task.uuid, object: type})
      
      Axios()
      .put(
        'tasks/' +
          this.task.uuid +
          '/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug,
        {
          config_issue_type_id: type.id,
        }).then((response) => {
          this.task.type = type
        })

    },
    search() {
      this.searchLoading = true
      this.types = []
      this.currentPage = 1
      this.getTaskTypes()
    },
  }
}
</script>

<template>
<div>
  <span
    v-if="task.type !== null && !dropdown"
    class="badge mr-2 mb-1"
    :style="'color: ' + invertColor(task.type.color, true) + ';background:' + task.type.color">
    {{task.type.title}}
  </span>
  <b-dropdown 
    v-if="dropdown && authorize('tasks', 'create')" 
    ref="dropdown" 
    left 
    class="styled-dropdown no-secondary" 
    @shown="loadMore">
    <template v-slot:button-content>
      <span
        class="badge mr-2"
        :style="'color: ' + invertColor(task.type.color, true) + ';background:' + task.type.color">
        {{task.type.title}}
        <font-awesome-icon 
        :icon="['far', 'angle-down']" 
        style="position:relative; margin-left:4px;top: 0px;" />
      </span>
    </template>
    <b-dropdown-form style="width:300px;">
      <b-input-group class="mb-1">
        <b-input-group-append class="wd-100">
          <b-form-input 
          v-model="searchTerm" 
          :placeholder="$t('Search type')"
          type="search" 
          autocomplete="off"
          size="sm"></b-form-input>
          <ButtonLoading
          :loading="searchLoading"
          type="btn-sm"
          mode="button"
          icon="search"
          @action="search"
          ></ButtonLoading>
        </b-input-group-append>
      </b-input-group>
    </b-dropdown-form>
    <div class="scroll-activate-h200 label-line">
      <b-link v-for="type in types" 
        :key="type.id" 
        class="dropdown-item project-label-line" 
        @click="changeTaskType(type)">
        <div class="square" :style="'background:' + type.color"></div>
        <span class="font-weight-bold">{{ type.title }}</span>
      </b-link>
    </div>
  </b-dropdown>
</div>
</template>
