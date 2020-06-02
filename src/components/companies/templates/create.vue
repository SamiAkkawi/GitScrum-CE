<script>
import Axios from '@utils/axios'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: { ButtonLoading},
  props: {
    template: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      name: '',
      loading: false,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
    }
  },
  methods: {
    createTemplate() {
      this.loading = true
      Axios()
        .post('templates/' + this.template + '/?company_slug=' + this.currentCompany.slug, {
          name: this.name,
        })
        .then((response) => {
          this.loading = false
          this.name = ''
          this.$emit('create', response.data.data)
        })
        .catch((e) => {
          this.loading = false
        })
    },
  },
}
</script>

<template>
  <b-card :header="$t('Create a Template')">
    <b-card-text>
      <b-input-group>
      <b-input-group-append>
        <b-form-input 
        v-model="name" 
        :placeholder="$t('Template name')"
        type="text" size="sm"></b-form-input>
        <ButtonLoading
        :loading="loading"
        type="btn-sm"
        icon="plus"
        @action="createTemplate"
        ></ButtonLoading>
      </b-input-group-append>
      </b-input-group>
    </b-card-text>
  </b-card>
</template>
