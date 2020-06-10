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
        effort: 0,
      }
    },

    getEndpoint(){

      let url = 'templates/effort/' + 
        this.templateSelected.id + 
        '/items/?company_slug=' + 
        this.currentCompany.slug

      if ( this.$route.params.projectSlug ) {
        url = 'project-templates/effort/?company_slug=' +
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
  <b-card :title="$t('Create a new Effort')" class="card-body-create">
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
  </b-card>
</template>
