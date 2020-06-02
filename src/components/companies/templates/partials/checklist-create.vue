<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: { ButtonLoading },
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
      type: String,
      required: false,
      default: '',
    },
    checklistItem: {
      type: Boolean,
      required: false,
      default: false,
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
      }
    },

    getEndpoint(){

      let url = 'templates/checklist/' + 
        this.templateSelected.id + 
        '/items/?company_slug=' + 
        this.currentCompany.slug

      if ( this.$route.params.projectSlug ) {
        url = 'project-templates/checklist/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
      }

      return url

    },

    addItem() {
      let url = this.getEndpoint()
      this.loading = true

      let params = {
        title: this.item.title,
      }

      if (this.item) {
        this.$set(params, 'parent_id', this.templateSelected.id)
      }

      Axios()
        .post(url, params)
        .then((response) => {
          this.loading = false
          let item = response.data.data
          if (item.parent_id !== null) {
            this.$delete(item, 'items')
          }

          this.templateSelected.items.push(item)
          this.item = this.defaultItem()
        })
    },
  },
}
</script>

<template>
  <b-card :title="$t('Create a new Checklist Item')" class="card-body-create">
    <b-input-group>
      <b-input-group-append>
        <b-form-input 
        v-model="item.title" 
        :placeholder="$t('Checklist item name')"
        type="text" 
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