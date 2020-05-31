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

    addItem() {
      this.loading = true
      Axios()
        .post('templates/workflow/' + this.templateSelected.id + '/items/?company_slug=' + this.currentCompany.slug, {
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
      v-model="item.title" 
      :placeholder="$t('Stage name')"
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
