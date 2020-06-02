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
      loading: false,
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
          type: this.item.type,
          meta: this.item.meta,
        }
      }

      return {
        name: this.item.title,
        type: this.item.type,
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
      this.loading = true
      let url = this.getUrl()

      console.log(this.prepareTaskCustomFieldObject())

      Axios()
        .post(url, this.prepareTaskCustomFieldObject())
        .then((response) => {
          this.loading = false
          let item = response.data.data
          this.appendParam(item)
          this.templateSelected.items.push(this.prepareType(item))

          this.item = this.defaultItem()
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

      this.item.optionCustomFieldSelect.splice(
        this.item.optionCustomFieldSelect.indexOf(option),
        1
      )

    },
  },
}
</script>

<template>
  <div>
    <b-input-group>
      <b-input-group-append>
        <b-form-select 
        v-model="item.type" 
        :options="customFieldOptions" 
        size="sm" 
        class="mr-2" 
        @input="checkSelected" ></b-form-select>
        <b-form-input 
        v-model="item.title" 
        :placeholder="$t('Custom Field item name')"
        type="text" 
        maxlength="25"
        size="sm"></b-form-input>
        <ButtonLoading
        v-if="item.type !== 'select'"
        :loading="loading"
        type="btn-sm"
        icon="plus"
        @action="addTaskCustomField"></ButtonLoading>
      </b-input-group-append>
    </b-input-group>

    <b-row v-if="item.type === 'select'">
      <b-col class="col-12 mt-2">
        <span>{{ $t('Add items to selectable option') }}</span>
        <b-input-group>
          <b-input-group-append>
            <b-form-input 
            v-model="newSelectableOptionName" 
            :placeholder="$t('Selectable option name')"
            type="text" 
            maxlength="25"
            size="sm"></b-form-input>
            <ButtonLoading
            :loading="loading"
            type="btn-sm"
            icon="plus"
            @action="addSelectableOption"
            ></ButtonLoading>
          </b-input-group-append>
        </b-input-group>
      </b-col>
    </b-row>
    <b-row v-if="item.type === 'select'">
      <b-col class="col-12 mt-2">
        <b-card v-if="item.optionCustomFieldSelect.length">
          <b-input-group v-for="option in item.optionCustomFieldSelect" :key="option.id" class="mb-2">
            <b-input-group-append>
              <b-form-input 
              v-model="option.name" 
              :placeholder="$t('Selectable option name')"
              type="text" 
              maxlength="25"
              size="sm"></b-form-input>
              <ButtonLoading
              :loading="loading"
              type="btn-sm"
              icon="minus"
              @action="delSelectableOption(option)"></ButtonLoading>
            </b-input-group-append>
          </b-input-group>
        </b-card>
        <ButtonLoading
          :loading="loading"
          type="btn-md"
          :title="$t('Add Custom Field')"
          @action="addTaskCustomField"></ButtonLoading>
      </b-col>
    </b-row>
  </div>
</template>
