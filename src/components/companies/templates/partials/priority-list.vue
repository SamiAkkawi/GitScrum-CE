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
  created() {
    if ( !this.templateSelected.items ){
      this.templateSelected.items = this.templateSelected
    }
  },
  methods: {
    getEndpoint(id){

      let url = 'templates/priority/' + 
        this.templateSelected.id + 
        '/items/' + id + '/?company_slug=' + 
        this.currentCompany.slug

      if ( this.$route.params.projectSlug ) {
        url = 'project-templates/priority/' + id + '/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
      }

      return url

    },

    updateItem(params, id) {
      let url = this.getEndpoint(id)
      Axios()
        .put(url, params)
        .then((response) => {})
    },

    deleteItem(item) {
      let url = this.getEndpoint(item.id)
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
              {{ $t('Default Priority') }} 
              <font-awesome-icon 
                v-b-popover.hover.top="$t('Whenever you create a task without choosing a user story priority. This will be the default user story priority.')" 
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
          :placeholder="$t('Priority name')"
          type="text" 
          maxlength="25"
          size="sm"
          style="border-radius:4px !important"
          class="mr-1"
          @change="updateItemTitle($event, item.id)"></b-form-input>
        </b-input-group>
      </b-card-text>
    </b-card>
  </Draggable>
</template>
