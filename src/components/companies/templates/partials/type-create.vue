<script>
import Axios from '@utils/axios'
import Swatches from 'vue-swatches'
import ButtonLoading from '@components/utils/button-loading'
import 'vue-swatches/dist/vue-swatches.min.css'

export default {
  components: { Swatches, ButtonLoading },
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
  },
  data() {
    return {
      item: this.defaultItem(),
      loading: false,
    }
  },
  methods: {
    defaultItem() {
      return {
        id: null,
        title: '',
        color: '#9900ff',
        blocked: false,
      }
    },

    getUrl() {
      let url = ''

      if (this.projectSlug) {
        url += 'project-'
      }

      url += 'templates/type/'

      if (!this.projectSlug) {
        url += this.templateSelected.id + '/items/'
      }

      url += '?company_slug=' + this.currentCompany.slug

      if (this.projectSlug) {
        url += '&project_slug=' + this.projectSlug
      }

      return url
    },

    addItem() {
      this.loading = true
      let url = this.getUrl()
      Axios()
        .post(url, {
          code: this.item.code,
          title: this.item.title,
          color: this.item.color,
          type: 'issues',
        })
        .then((response) => {
          this.loading = false
          this.templateSelected.items.push(response.data.data)
          this.item = this.defaultItem()
        })
    },
  },
}
</script>

<template>
  <b-input-group>
    <Swatches 
      v-model="item.color" 
      colors="text-advanced" 
      class="swatches-input" 
      popover-to max-height="400"></Swatches>
    <b-input-group-append>
      <b-form-input 
      v-model="item.code" 
      :placeholder="$t('Task type code')"
      type="text" 
      style="width:150px;border-radius:4px !important;"
      class="mr-2"
      maxlength = "4"
      size="sm"></b-form-input>
      <b-form-input 
      v-model="item.title" 
      :placeholder="$t('Task type name')"
      type="text" 
      maxlength="25"
      size="sm"></b-form-input>
      <ButtonLoading
      :loading="loading"
      type="btn-sm"
      icon="plus"
      @action="addItem"
      ></ButtonLoading>
    </b-input-group-append>
  </b-input-group>
</template>
