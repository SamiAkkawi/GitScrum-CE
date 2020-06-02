<script>
import Axios from '@utils/axios'
import Draggable from 'vuedraggable'
import Swatches from 'vue-swatches'
import vSelect from 'vue-select'

import 'vue-swatches/dist/vue-swatches.min.css'
import 'vue-select/dist/vue-select.css'

export default {
  components: { Draggable, Swatches, vSelect },
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
  watch: {
    templateSelected() {
      this.convertItemOptions()
    },
  },
  created() {
    this.convertItemOptions()
  },
  data() {
    return {
      stateOptions: [
        {
          value: 0,
          text: 'TODO',
        },
        {
          value: 2,
          text: 'In Progress',
        },
        {
          value: 1,
          text: 'Done',
        },
      ],
      emailExpression: /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,24}))$/,
    }
  },
  methods: {
    convertItemOptions() {
      this.templateSelected.items.forEach((item) => {
        item = this.setItemState(item)
        if (item.emails) {
          item.emails = item.emails.split(',')
        }
        this.$set(item, 'emailError', '')
      })
    },

    setItemState(item) {
      const s = this.stateOptions.find((s) => s.value === item.state)
      item.state = {
        value: item.state,
        name: s.name,
      }
      return item
    },

    updateItem(params, id) {
      Axios()
        .put(
          'templates/workflow/' +
            this.templateSelected.id +
            '/items/' +
            id +
            '/?company_slug=' +
            this.currentCompany.slug,
          params
        )
        .then((_) => {})
        .catch((e) => {
          console.error(e)
        })
    },

    deleteItem(item) {

      this.$bvModal.msgBoxConfirm(this.$t('Do you really want to delete?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          Axios()
          .delete(
            'templates/workflow/' +
              this.templateSelected.id +
              '/items/' +
              item.id +
              '/?company_slug=' +
              this.currentCompany.slug
          )
          .then((response) => {
            this.templateSelected.items.splice(this.templateSelected.items.indexOf(item), 1)
          })
          .catch((e) => {
            console.error(e)
          })
        }
      })

    },

    updateItemPosition(item) {
      this.updateItem( { position: item.moved.newIndex + 1 }, item.moved.element.id )
    },

    updateItemDefault(value, id) {
      this.updateItem( { default: value }, id )
    },

    updateItemColor(color, id) {
      this.updateItem( { color: color }, id )
    },

    updateItemTitle(title, id) {
      if (title.length) {
        this.updateItem( { title: title }, id )
      }
    },

    updateState(state, id) {
      this.updateItem( { state: state }, id )
    },

    validateEmail: function(email) {
      return this.emailExpression.test(email)
    },

    updateItemEmails(emails, item) {
      item.emailError = ''
      emails.forEach((email) => {
        if (!this.validateEmail(email)) {
          item.emailError = email
          emails.splice(emails.indexOf(email))
        }
      })

      if (!item.emailError.length) {
        this.updateItem( {emails: emails.join(),} , item.id )
      }
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
            <b-form-radio
              v-model="item.default"
              class="txt-68748F"
              value="true"
              name="typeDefault"
              @change="updateItemDefault(1, item.id)">
              {{ $t('Default Stage') }} 
              <font-awesome-icon 
                v-b-popover.hover.top="$t('Whenever you create a task without choosing a workflow stage. This will be the default stage.')" 
                :icon="['fal', 'question-circle']" 
                class="ml-2" />
            </b-form-radio>
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
          <Swatches 
          v-model="item.color" 
          colors="text-advanced" 
          class="swatches-input" 
          popover-to max-height="400"
          @close="updateItemColor($event, item.id)"></Swatches>
          <b-form-input 
          v-model="item.title" 
          :placeholder="$t('Stage name')"
          type="text" 
          maxlength="25"
          size="sm"
          style="border-radius:4px !important"
          class="mr-1"
          @change="updateItemTitle($event, item.id)"></b-form-input>
          <b-form-select 
          v-model="item.state.value" 
          :options="stateOptions" 
          size="sm" 
          @change="updateState($event, item.id)"></b-form-select>
        </b-input-group>
        <label for="tags-separators">{{ $t('Enter emails to receive notifications when the task is in this stage') }}</label>
        <b-form-tags
          v-model="item.emails"
          input-id="tags-separators"
          separator=" ,;"
          placeholder="Enter emails"
          no-add-on-enter
          class="mb-1 mt-1"
          tag-variant="dark"
          @input="updateItemEmails($event, item)"></b-form-tags>
        <small>{{ $t('Enter tags separated by space, comma or semicolon') }}</small>
      </b-card-text>
    </b-card>
  </Draggable>
</template>
