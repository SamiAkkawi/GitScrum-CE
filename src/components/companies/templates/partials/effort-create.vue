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
        effort: 0,
      }
    },

    getUrl() {
      let url = ''

      if (this.projectSlug) {
        url += 'project-'
      }

      url += 'templates/effort/'

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
          title: this.item.title,
          effort: this.item.effort,
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
    <b-input-group-append>
      <b-form-input 
      v-model="item.effort" 
      :placeholder="$t('Task effort points')"
      type="number" 
      style="width:160px;border-radius:4px !important;"
      class="mr-2"
      maxlength="6"
      size="sm"></b-form-input>
      <b-form-input 
      v-model="item.title" 
      :placeholder="$t('Task effort name')"
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
