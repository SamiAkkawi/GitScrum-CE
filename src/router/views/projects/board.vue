<script>
import Layout from '@layouts/tpl-main-large'
import Axios from '@utils/axios'
import BoardTask from '@components/projects/board-task'
import Draggable from 'vuedraggable'
import { isMobile } from 'mobile-device-detect'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'
import TitleLoading from '@components/utils/title-loading'
import ShareableBoard from '@layouts/partials/shareable-board'
import Invite from '@components/projects/team-members/invite'
import ButtonLoading from '@components/utils/button-loading'

export default {
  page: {
    title: 'Board',
    meta: [{ name: 'description', content: '' }],
  },
  components: {
    Layout,
    TitleLoading,
    BoardTask,
    Draggable,
    Swatches,
    ShareableBoard,
    Invite,
    ButtonLoading
  },
  data() {
    return {
      loading: true,
      btnLoading: false,
      canDragColums: isMobile,

      width: 0,
      newColumn: '',
      colorColumn: '#000000',
      workflows: [],
      boxNewColumn: 'plus',
      nameColumn: '',

      btnOptionSelected: '0',
      btnOptions: [
        { text: 'All Tasks', value: '0' },
        { text: 'Archived Tasks', value: '1' }
      ],

      cardColors: false,
      
    }
  },
  computed: {
    dragOptions() {
      return {
        animation: 0,
        group: 'columns',
        disabled: false,
        ghostClass: 'ghost',
      }
    },
  },
  watch: {
    btnOptionSelected() {
      this.$store.dispatch('tasksArchived/setIsArchived', this.btnOptionSelected)
      this.$eventBus.$emit('reload-column', false )
    }
  },
  mounted() {
    this.getWorkflows()
  },
  methods: {
    getWorkflows() {
      Axios()
        .get(
          'projects-workflows/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          this.workflows = response.data.data
          this.width = this.workflows.length * 295 + 280
          this.loading = false
        })
        .catch((e) => {
          console.error(e)
        })
    },
    moveColumn(evt) {
      Axios()
        .put(
          '/projects-workflows/' +
            evt.moved.element.id +
            '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug,
          {
            position: evt.moved.newIndex + 1,
          }
        )
        .then((response) => {})
        .catch((e) => {
          console.error(e)
        })
    },
    addNewColumn() {
      var nameColumn = this.nameColumn
      if (nameColumn.length > 3) {
        this.btnLoading = true

        Axios()
          .post(
            '/projects-workflows/?company_slug=' +
              this.$route.params.companySlug +
              '&project_slug=' +
              this.$route.params.projectSlug,
            {
              title: this.nameColumn,
              color: this.colorColumn,
              status: 0,
            }
          )
          .then((response) => {
            this.btnLoading = false
            this.toggleNewColumn()
            this.workflows.push(response.data.data)
            this.width += 295
            this.nameColumn = ''
            this.colorColumn = '#000000'
          })
          .catch((e) => {
            console.error(e)
          })
      } else {
        alert(this.$t('Column name must have at least 4 characters'))
      }
    },
    toggleNewColumn() {
      if (this.boxNewColumn === 'minus') {
        this.boxNewColumn = 'plus'
      } else{
        this.boxNewColumn = 'minus'
      }
    },
  },
}
</script>

<template>
  <Layout>
    
    <template slot="header-left">
      <TitleLoading :title="$t('Board')" :loading="loading"></TitleLoading>
    </template>

    <template slot="header-right">
      <b-button-toolbar id="header-navbar-project" aria-label="Toolbar with button groups and dropdown menu">
        <b-form-radio-group
          v-model="btnOptionSelected"
          :options="btnOptions"
          buttons
          name="radios-btn-default"></b-form-radio-group>
        <b-button-group class="mx-1">
           <b-button v-b-toggle.sidebar-task-filter>
              <font-awesome-icon :icon="['far', 'filter']"/>
              {{ $t('Advanced Filters') }}
            </b-button>
        </b-button-group>
        <ShareableBoard v-if="authorize('header', 'share')" class="ml-2 mr-1"></ShareableBoard>
        <Invite></Invite>
      </b-button-toolbar>
    </template>

    <div slot="content">
      <div class="lists" :style="'width:' + width + 'px'">
        <Draggable
          :disabled="canDragColums"
          class="list row"
          style="min-width: calc(100% + 40px);"
          tag="ul"
          :list="workflows"
          v-bind="dragOptions"
          draggable=".draggableColumn"
          @change="moveColumn">
          <li
            v-for="workflow in workflows"
            :key="workflow.id"
            class="draggableColumn"
            style="min-height: calc(100vh - 120px);">
            <BoardTask :workflow="workflow"></BoardTask>
          </li>
          <div v-show="!loading" class="mg-r-20 mg-l-0">
            <div class="addColumnBox" @click="toggleNewColumn">
              <font-awesome-icon :icon="['fas', boxNewColumn]" />
            </div>
            <b-card v-show="boxNewColumn === 'minus'" :title="$t('Create a new Workflow Stage')" class="card-body-create mt-2">
              <b-input-group>
                <b-input-group-append>
                  <Swatches 
                  v-model="colorColumn" 
                  colors="text-advanced" 
                  class="swatches-input" 
                  popover-to max-height="400"></Swatches>
                  <b-form-input
                  v-model="nameColumn"
                  size="sm"
                  :placeholder="$t('Column name')" />
                  <ButtonLoading
                  :loading="btnLoading"
                  type="btn-sm"
                  icon="plus"
                  @action="addNewColumn"></ButtonLoading>
                </b-input-group-append>
              </b-input-group>
            </b-card>
          </div>
        </Draggable>
      </div>
    </div>
  </Layout>
</template>
