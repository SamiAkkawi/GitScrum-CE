<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: { ButtonLoading },
  props: {
    templateSelected: {
      type: Object,
      required: true,
    },
    currentCompany: {
      type: Object,
      required: true,
    },
    projectSlug: {
      type: String,
      required: false,
      default: null,
    },
  },
  data() {
    return {
      btnLoading: false,
      newSelectableOptionName: '',
      item: this.defaultItem(),
      customFieldOptions : [
        { text: 'Small text', value: 'text' },
        { text: 'Long text', value: 'textarea' },
        { text: 'Checkbox', value: 'checkbox' },
        { text: 'Selectable options', value: 'select' },
      ]
    }
  },
  methods: {
    defaultItem() {
      return {
        id: null,
        title: '',
        type: 'text',
        optionCustomFieldSelectable: false,
        optionCustomFieldSelect: [],
        meta: '',
        popoverShow: false,
      }
    },

    buildMeta() {
      let meta = []
      this.item.optionCustomFieldSelect.forEach((opt) => {
        meta.push(opt.name)
      })
      this.item.meta = meta.join(',')
    },

    prepareType(item) {
      for (let i = 0; i < this.customFieldOptions.length; i++) {
        if (item.type === this.customFieldOptions[i].value) {
          item.type = this.customFieldOptions[i]
          break
        }
      }
      return item
    },

    prepareTaskCustomFieldObject() {
      if (this.item.meta.length) {
        return {
          name: this.item.title,
          type: this.item.type.code,
          meta: this.item.meta,
        }
      }

      return {
        name: this.item.title,
        type: this.item.type.code,
      }
    },

    getUrl() {
      let url = ''

      if (this.projectSlug) {
        url += 'project-'
      }

      url += 'templates/field/'

      if (!this.projectSlug) {
        url += this.templateSelected.id + '/items/'
      }

      url += '?company_slug=' + this.currentCompany.slug

      if (this.projectSlug) {
        url += '&project_slug=' + this.projectSlug
      }

      return url
    },

    addTaskCustomField() {
      if (
        this.item.optionCustomFieldSelectable &&
        this.item.optionCustomFieldSelect.length
      ) {
        this.buildMeta()
      }
      this.btnLoading = true
      let url = this.getUrl()

      Axios()
        .post(url, this.prepareTaskCustomFieldObject())
        .then((response) => {
          this.btnLoading = false
          let item = response.data.data
          this.appendParam(item)
          this.templateSelected.items.push(this.prepareType(item))

          this.item = this.defaultItem()
        })
        .catch((e) => {
          this.btnLoading = false
          // console.error(e)
        })
    },

    appendParam(item) {
      let optionArray = []
      if (this.projectSlug) {
        optionArray = this.appendProjectParam(item)
      } else {
        optionArray = this.appendCompanyParam(item)
      }

      this.$set(item, 'optionCustomFieldSelect', optionArray)
    },

    appendCompanyParam(item) {
      let optionArray = []
      if (item.meta && item.meta !== null && item.meta.length) {
        optionArray = item.meta.split(',')
        optionArray.forEach((opt, key) => {
          optionArray[key] = {
            id: opt,
            name: opt,
          }
        })
      }
      return optionArray
    },

    appendProjectParam(item) {
      let optionArray = []

      if (item.meta && item.meta !== null) {
        optionArray = Object.values(item.meta)
        optionArray.forEach((opt, key) => {
          optionArray[key] = {
            id: opt,
            name: opt,
          }
        })
      }

      return optionArray
    },

    checkSelected(evt) {
      if (evt.code === 'select') {
        this.item.optionCustomFieldSelectable = true
      } else {
        this.item.optionCustomFieldSelectable = false
      }
    },

    addSelectableOption() {
      if (this.newSelectableOptionName.length) {
        let option = {
          id: null,
          name: this.newSelectableOptionName,
        }

        this.item.optionCustomFieldSelect.push(option)
        this.newSelectableOptionName = ''
      }
    },

    delSelectableOption(option) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.item.optionCustomFieldSelect.splice(
            this.item.optionCustomFieldSelect.indexOf(option),
            1
          )
        }

      })
    },
  },
}
</script>

<template>
  <div class="row">
    <div class="col-md-12">
      <div class="create-template-item-div">
        <h5 class="fw-500 tx-14-px txt-001737 title m-0">
          {{ $t('Create a Task Custom Field') }}
        </h5>
        <div class="item-input d-flex mt-10-px">
          <input
            v-model="item.title"
            pattern=".{3,}"
            required
            type="text"
            class="form-control mr-5-px"
            :placeholder="$t('Custom Field item name')"
          />
          <div class="v-select-sm ml-5-px only-left-border-radius">
            <b-form-select v-model="item.type" :options="customFieldOptions" @input="checkSelected"></b-form-select>
            
          </div>
          <div class="input-group-append">
            <button
              v-if="!btnLoading"
              class="btn btn-sm btn-primary"
              type="button"
              :disabled="!item.title || !item.type"
              @click="addTaskCustomField"
            >
              <font-awesome-icon :icon="['fa', 'plus']" />
            </button>

            <button v-if="btnLoading" class="btn btn-sm btn-primary" type="button">
              <b-spinner :label="$t('Loading')" small class="title-loading"></b-spinner>
            </button>
          </div>
        </div>
        <div v-if="item.optionCustomFieldSelectable">
          <div class="selectable-options mt-20-px">
            <div class="selectable-options-create">
              <div class="item-input d-flex">
                <input
                  v-model="newSelectableOptionName"
                  class="form-control mr-15-px"
                  :placeholder="$t('Option label name')"
                  type="text"
                />
                <font-awesome-icon
                  :icon="['far', 'plus-circle']"
                  class="txt-464DEE"
                  style="height: 32px; cursor: pointer;"
                  @click="addSelectableOption"
                />
              </div>
            </div>
            <div class="selectable-options-list">
              <div v-if="item.optionCustomFieldSelect.length">
                <span class="txt-68748F tx-12-px fw-500 sub-title">
                  {{ $t('Options List') }}
                </span>
                <hr class="m-0 mt-10-px" />
                <div v-for="option in item.optionCustomFieldSelect" :key="option.id">
                  <div class="item-input d-flex mt-10-px">
                    <input
                      v-model="option.name"
                      class="form-control mr-15-px"
                      :placeholder="$t('Option label name')"
                      type="text"
                    />
                    <font-awesome-icon
                      @click="delSelectableOption(option)"
                      :icon="['far', 'minus-circle']"
                      class="txt-A2B3CA"
                      style="height: 32px; cursor: pointer;"
                    />
                  </div>
                </div>
              </div>
              <div v-else class="alert alert-info m-0">
                {{ $t('List without options') }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
