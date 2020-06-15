<script>
import Axios from '@utils/axios'

export default {
  data() {
    return {
      data: [],
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
    }
  },
  created() {
    this.getWhiteLabel()
  },
  methods: {
    getWhiteLabel() {
      let domain = window.location.host
      
      let companySlug = this.currentCompany ? this.currentCompany.slug : null
      
      let whiteLabel = this.$store.getters['whiteLabel/getWhiteLabel']

      if ( !whiteLabel ){

        Axios()
          .get('whitelabel-domain/?domain=' + domain + '&company_slug=' + companySlug)
          .then((response) => {
            this.data = response.data.data.branding
            if (!this.data.name) {
              this.$router.push({ path: '/ooops' })
              return
            }
            this.$store.dispatch('whiteLabel/setWhiteLabel', this.data)
          })

      } else {
        this.data = whiteLabel
      }
      
    },
    gaClickLogo(){
      this.$ga.event('header_bar', 'click_logo')
      this.$router.push({ name: 'workspaces.projects' })
    }
  },
}
</script>

<template>
  <b-navbar-brand tag="li">
    <span class="cursor-pointer" @click="gaClickLogo">
      <img id="companyLogo" style="max-width: 122px; max-height: 22px; margin-left: 8px;" :src="data.logo_header" :alt="data.name" :title="data.name" />
    </span>
  </b-navbar-brand>
</template>
