<script>
import Axios from '@utils/axios'
import Draggable from 'vuedraggable'

export default {
  name: 'TaskChecklistList',
  components: { Draggable },
  props: {
    currentCompany: {
      type: Object,
      required: true,
    },
    templateSelected: {
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
    return {}
  },
  methods: {
    getUrl(id) {
      let url = ''

      if (this.projectSlug) {
        url += 'project-'
      }

      url += 'templates/checklist/'

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

    updateItem(params, id) {
      let url = this.getUrl(id)
      Axios()
        .put(url, params)
        .then((response) => {})
        .catch((e) => {})
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
        }
      })

    },

    updateItemTitle(title, id) {
      this.updateItem( { title: title }, id )
    },

    updateItemPosition(item) {
      this.updateItem( { position: item.moved.newIndex + 1 }, item.moved.element.id )
    },
  },
}
</script>

<template>
  <Draggable v-if="templateSelected.items.length" v-model="templateSelected.items" tag="div" ghost-class="ghost" @change="updateItemPosition($event)">
    <b-card v-for="item in templateSelected.items" :key="item.id">
      <template v-slot:header>
        <b-row class="cursor-grab">
          <b-col cols="9">
            <strong>{{ $t('Checklist Item') }}</strong>
          </b-col>
          <b-col cols="3" class="text-right">
            <a href="javascript:;" class="card-delete-hover" @click="deleteItem(item)">
              <span>{{ $t('Delete') }}</span>
            </a>
          </b-col>
        </b-row>
      </template>
      <b-card-text>
        <b-form-input v-model="item.title" @change="updateItemTitle($event, item.id)"></b-form-input>
      </b-card-text>
    </b-card>
  </Draggable>
</template>