<script>
import Axios from '@utils/axios'
import { modalManager } from '@state/helpers'
import ButtonLoading from '@components/utils/button-loading'

export default {
  components: {ButtonLoading},
  data() {
    return {
      loading: false,
      btnLoading: false,
      currentCompany: JSON.parse(localStorage.getItem('CURRENT_COMPANY')),
      companies: JSON.parse(localStorage.getItem('COMPANIES_USER')),
      companyName: ''
    }
  },
  computed: {
    ...modalManager,
  },
  watch: {
    statusModal(object) {
      if (object.item.name === 'createCompany') {
        object.item.name = ''
        this.$refs['modal'].show()
      }
    },
  },
  methods: {
    hideModal() {
      this.closeModal(this.$refs['modal'])
    },
    toggleModal() {
      this.$refs['modal'].toggle('#toggle-btn')
    },
    create(){
      if(this.companyName.length === 0){
        return alert(this.$t('Company name is mandatory'))
      }

      this.loading = true
      Axios()
        .post('companies', {name: this.companyName})
        .then(response => {
          this.resetData()
          this.companies.push(response.data.data)

          localStorage.setItem('CURRENT_COMPANY', JSON.stringify(response.data.data))
          localStorage.setItem('COMPANIES_USER', JSON.stringify(this.companies))
          location.href = '/projects'
        }).catch(error => {
          this.resetData()
          return alert(error.data.message)
        })
    },
    resetData(){
      this.loading = false
      this.companyName = ''
    }
  },
}
</script>

<template>
  <b-modal id="modal-black" ref="modal" hide-header hide-footer size="xx">
    <b-container>
      <b-row class="mb-3">
        <b-col>
          <h3 class="mb-0">{{ $t('Create another company') }}</h3>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-form-group
            :label="$t('New company name')">
            <b-form-input 
            v-model="companyName" 
            type="text" 
            maxlength="45" trim></b-form-input>
          </b-form-group>
        </b-col>
      </b-row>
      <b-row class="mt-2">
        <b-col cols="4" align-self="center">
          <b-link 
            v-show="!loading" 
            @click="hideModal" 
            v-text="$t('Cancel')" />
        </b-col>
        <b-col class="text-right">
          <ButtonLoading
            :loading="loading"
            :title="$t('Create New Company')"
            :title-loading="$t('Creating')"
            type="btn-md"
            mode="button"
            @action="create"></ButtonLoading>
        </b-col>
      </b-row>
    </b-container>
  </b-modal>
</template>
