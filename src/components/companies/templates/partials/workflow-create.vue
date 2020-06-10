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
      type: Object | Array,
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

    getEndpoint(){

      let url = 'templates/workflow/' + 
        this.templateSelected.id + 
        '/items/?company_slug=' + 
        this.currentCompany.slug

      if ( this.$route.params.projectSlug ) {
        url = 'project-templates/workflow/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
      }

      return url

    },

    addItem() {
      this.loading = true
      let url = this.getEndpoint()
      Axios()
        .post(url, {
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
  <b-card :title="$t('Create a new Workflow Stage')" class="card-body-create">
    <b-input-group>
      <b-input-group-append>
        <Swatches 
        v-model="item.color" 
        colors="text-advanced" 
        class="swatches-input" 
        popover-to max-height="400"></Swatches>
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
  </b-card>
</template>
