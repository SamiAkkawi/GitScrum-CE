<script>
import Axios from '@utils/axios'
import Draggable from 'vuedraggable'
import Swatches from 'vue-swatches'
import 'vue-swatches/dist/vue-swatches.min.css'

export default {
  components: { Draggable, Swatches },
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

      url += 'templates/type/'

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
        .catch((e) => {
          console.error(e)
        })
    },

    deleteItem(item) {
      let url = this.getUrl(item.id)
      Axios()
        .delete(url)
        .then((response) => {
          this.templateSelected.items.splice(this.templateSelected.items.indexOf(item), 1)
        })
        .catch((e) => {
          console.error(e)
        })
    },

    updateItemColor(color, id) {
      this.updateItem(
        {
          color: color,
        },
        id
      )
    },

    updateItemDefault(value, id) {
      this.updateItem(
        {
          default: value,
        },
        id
      )
    },

    updateItemTitle(title, id) {
      this.updateItem(
        {
          title: title,
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

    updateItemCode(item, id) {
      this.updateItem(
        {
          code: item,
        },
        id
      )
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
              {{ $t('Default Task Type') }} 
              <font-awesome-icon 
                v-b-popover.hover.top="$t('Whenever you create a task without choosing a task type. This will be the default task type.')" 
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
          <b-input-group-append>
            <b-form-input
            v-model="item.code"
            style="border-radius:4px !important"
            maxlength="4"
            autocomplete="false"
            size="sm"
            class="mr-1"
            @change="updateItemCode($event, item.id)"></b-form-input>
            <b-form-input 
            v-model="item.title" 
            :placeholder="$t('Stage name')"
            type="text" 
            maxlength="25"
            size="sm"
            style="border-radius:4px !important"
            class="mr-1"
          @change="updateItemTitle($event, item.id)"></b-form-input>
          </b-input-group-append>
        </b-input-group>
      </b-card-text>
    </b-card>
  </Draggable>
</template>
