<script>
import Axios from '@utils/axios'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: {
    Swatches,
    ButtonLoading
  },
  props: {
    project: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      search: '',
      btnLoading: false,
      selectableLabels: [],
      newLabel: this.defaultLabel(),
      labelColorStyleProps: {
        blank: true,
        width: 15,
        height: 15,
        class: 'm1',
      },
      currentEditingLabel: {
        title: '',
        color: '',
      },
      activeTab: 0,
      tabs: ['myLabels', 'searchLabels'],
      createBtnLoading: false,
      searchBtnLoading: false,
    }
  },
  watch: {
    project() {
      this.activeTab = 0
    },
  },
  methods: {
    defaultLabel() {
      return {
        id: null,
        title: '',
        color: '#333333'
      }
    },

    searchLabels() {
      this.searchBtnLoading = true
      Axios()
        .get(
          'projects-labels/not-added/?company_slug=' +
            this.project.company.slug +
            '&project_slug=' +
            this.project.slug +
            '&search=' +
            this.search
        )
        .then((response) => {
          this.searchBtnLoading = false
          this.selectableLabels = response.data.data
        })
        .catch((e) => {
          console.error(e)
        })
    },

    onClickSearchLabels() {
      this.searchLabels()
    },

    editLabel(label) {
      this.currentEditingLabel.color = label.color
      this.currentEditingLabel.title = label.title
    },

    updateLabel(label) {

      if (!label.title) {
        label.color = this.currentEditingLabel.color
        label.title = this.currentEditingLabel.title
        return
      }

      Axios()
        .put(
          'projects-labels/' +
            label.slug +
            '?company_slug=' +
            this.project.company.slug +
            '&project_slug=' +
            this.project.slug,
          {
            title: label.title,
            color: label.color,
          }
        )
        .then((_) => {
          this.updateLocalLabel()
        })
        .catch((e) => {
          console.error(e)
        })
    },

    updateLocalLabel() {
      let labels

      switch (this.tabNameSelected) {
        case 'searchLabels':
          labels = this.selectableLabels
          break
        case 'myLabels':
          labels = this.project.labels
          break
        default:
          labels = null
          break
      }

      if (labels !== null) {
        labels.forEach((label) => {
          if (label.slug === this.currentEditingLabel.slug) {
            this.$set(label, 'color', this.currentEditingLabel.color)
            this.$set(label, 'title', this.currentEditingLabel.title)
          }
        })
      }
      this.tabNameSelected = ''
    },

    onClickDeleteLabel(label) {

      this.$bvModal.msgBoxConfirm(this.$t('Are you sure? It will remove the label permanently'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete(
            'projects-labels/' +
              label.slug +
              '?company_slug=' +
              this.project.company.slug +
              '&project_slug=' +
              this.project.slug
          )
          .then((_) => {
            this.selectableLabels.splice(
              this.selectableLabels.indexOf(label),
              1
            )
          })
          .catch((e) => {
            console.error(e)
          })
        }
      })
      
    },

    onClickRemoveLabel(label) {
      Axios()
        .delete(
          'projects-labels/' +
            label.slug +
            '/detach?company_slug=' +
            this.project.company.slug +
            '&project_slug=' +
            this.project.slug
        )
        .then((_) => {
          this.project.labels.splice(this.project.labels.indexOf(label), 1)
          this.selectableLabels.push(label)
        })
        .catch((e) => {
          console.error(e)
        })
    },

    addLabelToProject(label) {
      Axios()
        .post(
          'projects-labels/' +
            label.slug +
            '/attach?company_slug=' +
            this.project.company.slug +
            '&project_slug=' +
            this.project.slug,
          label
        )
        .then((response) => {
          this.selectableLabels.splice(this.selectableLabels.indexOf(label), 1)
          this.project.labels.push(label)
        })
        .catch((e) => {
          console.error(e)
        })
    },

    saveLabel() {
      this.btnLoading = true
      if (!this.newLabel.title) {
        return
      }

      Axios()
        .post(
          'projects-labels/?company_slug=' +
            this.project.company.slug +
            '&project_slug=' +
            this.project.slug,
          this.newLabel
        )
        .then((response) => {
          this.btnLoading = false
          this.project.labels.push(response.data.data)
          this.search = ''
          this.newLabel = this.defaultLabel()
        })
        .catch((e) => {
          console.error(e)
        })
    },
  },
}
</script>

<template>
  <div class="manage-labels">
      <b-tabs v-model="activeTab" justified>
        <b-tab :title="$t('Project Labels')" active>
          <div  class="project-label-line label-create mt-3">
            <label class="label-for-input">Create a new label</label>
            <b-input-group>
              <Swatches
                  v-model="newLabel.color"
                  colors="text-advanced"
                  popover-to="right"
                  class="swatches-input"
                ></Swatches>
              <b-input-group-append>
                <b-form-input 
                v-model="newLabel.title" 
                :placeholder="$t('Label Name')"
                class="label-name"
                type="text" size="sm"></b-form-input>
                <ButtonLoading
                  :loading="btnLoading"
                  type="btn-sm"
                  @action="saveLabel"
                ></ButtonLoading>
              </b-input-group-append>
            </b-input-group>
          </div>

          <hr>

          <div v-for="label in project.labels" :key="label.id" class="project-label-line label-create">
            <b-input-group>
              <Swatches
                v-model="label.color"
                colors="text-advanced"
                popover-to
                class="swatches-input"
                @input="updateLabel(label)"
              ></Swatches>
              <b-input-group-append>
                <b-form-input 
                  v-model="label.title" 
                  :placeholder="$t('Label Name')"
                  type="text" size="sm"
                  @focus="editLabel(label)"
                  @blur="updateLabel(label)"></b-form-input>
                <b-button v-b-tooltip.hover title="Remove Label" variant="light" @click="onClickRemoveLabel(label)">
                  <font-awesome-icon :icon="['fas', 'times']" style="color:#1E1E2F"></font-awesome-icon>
                </b-button>
              </b-input-group-append>
            </b-input-group>
          </div>
        </b-tab>
        <b-tab  @click="searchLabels">
          <template v-slot:title>
            {{ $t('All Labels') }}&nbsp;&nbsp;<b-spinner v-if="searchBtnLoading" type="border" small></b-spinner> 
          </template>
          <div class="mt-3">
            <div v-for="label in selectableLabels" :key="label.id" class="project-label-line label-search">
              <b-input-group>
                <Swatches
                  v-model="label.color"
                  colors="text-advanced"
                  popover-to
                  class="swatches-input"
                  @input="updateLabel(label)"
                ></Swatches>
                <b-input-group-append>
                  <b-form-input 
                    v-model="label.title"
                    :placeholder="$t('Label Name')"
                    type="text" size="sm"
                    @focus="editLabel(label)"
                    @blur="updateLabel(label)"></b-form-input>
                  <b-button v-b-tooltip.top.hover title="Add Label" variant="light" @click="addLabelToProject(label)">
                    <font-awesome-icon :icon="['fas', 'plus']" style="color:#1E1E2F"></font-awesome-icon>
                  </b-button>
                  <b-button v-b-tooltip.hover title="Delete Label" variant="light" @click="onClickRemoveLabel(label)">
                    <font-awesome-icon :icon="['fas', 'trash']" style="color:#1E1E2F"></font-awesome-icon>
                  </b-button>
                </b-input-group-append>
              </b-input-group>
            </div>
          </div>
        </b-tab>
      </b-tabs>
  </div>
</template>
