<script>
import Axios from '@utils/axios'
import Draggable from 'vuedraggable'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: { Draggable, ButtonLoading },
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
    title: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      loading: false,
      newSelectableOptionName: [],
      customFieldOptions : [
        { text: 'Small text', value: 'text' },
        { text: 'Long text', value: 'textarea' },
        { text: 'Checkbox', value: 'checkbox' },
        { text: 'Selectable options', value: 'select' },
      ]
    }
  },
  mounted() {
    
    this.templateSelected.items.forEach((item) => {
      this.appendParam(item)
    })
  },
  methods: {
    getUrl(id) {
      let url = ''

      if (this.projectSlug) {
        url += 'project-'
      }

      url += 'templates/field/'

      if (!this.projectSlug) {
        url += this.templateSelected.id + '/items/'
      }

      if (id) {
        url += id + '/'
      }

      url += '?company_slug=' + this.currentCompany.slug

      if (this.projectSlug) {
        url += '&project_slug=' + this.projectSlug
      }

      return url
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
      if (item.meta && item.meta.length) {
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

    updateItem(params, id) {
      let url = this.getUrl(id)
      Axios()
        .put(url, params)
        .then((response) => {})
    },

    deleteItem(item) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          let url = this.getUrl(item.id)
          Axios()
          .delete(url)
          .then((response) => {
            this.templateSelected.items.splice(this.templateSelected.items.indexOf(item), 1)
          })
          .catch((e) => {
            console.error(e)
          })
        }
      })

    },

    updateItemName(name, id) {
      this.updateItem(
        {
          name: name,
        },
        id
      )
    },

    updateItemPosition(item) {
      this.updateItem(
        {
          position: item.moved.newIndex + 1,
        },
        item.moved.element.id
      )
    },

    updateItemType(type, item) {
      
      if (type === 'select') {
        this.$set(item, 'optionCustomFieldSelect', [])
      }

      this.updateItem( { type: type }, item.id )
    },

    updateSelectableOption(item) {
      
      let meta = this.buildMeta(item.optionCustomFieldSelect)
      this.updateItem( { meta: meta }, item.id )

    },

    addSelectableOption(item) {
      let option = {
        id: null,
        name: this.newSelectableOptionName[item.id],
      }

      if ( !item.optionCustomFieldSelect ){
        item.optionCustomFieldSelect = []
      }

      this.newSelectableOptionName[item.id] = ''
      item.optionCustomFieldSelect.push(option)
      this.updateSelectableOption(item)
    },

    delSelectableOption(item, option) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          item.optionCustomFieldSelect.splice(item.optionCustomFieldSelect.indexOf(option), 1)
          this.updateSelectableOption(item)
        }
      })

    },

    buildMeta(selectableOptions) {
      let meta = []
      selectableOptions.forEach((opt) => {
        meta.push(opt.name)
      })
      return meta.join(',')
    },

    getMeta(meta){
      if (meta){ 
        return meta.split(',')
      }
    }
  },
}
</script>

<template>
  <Draggable v-if="templateSelected.items.length" v-model="templateSelected.items" tag="div" ghost-class="ghost" @change="updateItemPosition($event)">
    <b-card v-for="item in templateSelected.items" :key="item.id">
      <template v-slot:header>
        <b-row class="cursor-grab">
          <b-col cols="9">
            <strong>{{ $t('Custom Field') }}</strong>
          </b-col>
          <b-col cols="3" class="text-right">
            <a href="javascript:;" class="card-delete-hover" @click="deleteItem(item)">
              <span>{{ $t('Delete') }}</span>
            </a>
          </b-col>
        </b-row>
      </template>
      <b-card-text>
        <b-input-group>
          <b-form-select 
          v-model="item.type" 
          :options="customFieldOptions" 
          size="sm" 
          class="mr-2" 
          @change="updateItemType($event, item)"></b-form-select>
          <b-form-input 
          v-model="item.name" 
          :placeholder="$t('Field name')"
          type="text" 
          maxlength="25"
          size="sm"
          style="border-radius:4px !important"
          class="mr-1"
          @change="updateItemName($event, item.id)"></b-form-input>
        </b-input-group>
        <b-input-group v-if="item.type === 'select'" class="mt-2">
          <b-form-input 
          v-model="newSelectableOptionName[item.id]" 
          :placeholder="$t('Selectable name')"
          type="text" 
          maxlength="25"
          size="sm"></b-form-input>
          <ButtonLoading
          :loading="loading"
          type="btn-sm"
          icon="plus"
          @action="addSelectableOption(item)"></ButtonLoading>
        </b-input-group>
        <b-row v-if="item.optionCustomFieldSelect && item.type === 'select'" align-v="center" class="mt-2">
          <b-col>
              <div class="border-bottom mt-2 mb-2">
                {{ $t('Selectable Options') }}
              </div>
              <b-input-group v-for="option in item.optionCustomFieldSelect" :key="option.id" class="mb-2">
                <b-form-input 
                v-model="option.name"
                :placeholder="$t('Selectable name')"
                type="text" 
                maxlength="25"
                size="sm"
                style="border-radius:4px !important"
                class="mr-2"
                @change="updateSelectableOption(item)"></b-form-input>
                <b-link @click="delSelectableOption(item, option)">
                  <font-awesome-icon :icon="['far', 'trash-alt']" />
                </b-link>
              </b-input-group>
          </b-col>
        </b-row>
      </b-card-text>
    </b-card>
  </Draggable>
</template>
