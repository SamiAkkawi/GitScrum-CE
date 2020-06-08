<script>
import Axios from '@utils/axios'

export default {
  props: {
    config: {
      type: Object | Array,
      required: true,
    },
    goto: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      configFormAnswer: [],
      loading: true,
      btnLoading: false,
      enableShareableLinkLoading: false,
      enableShareableLink: false,
      token: '',
      shareableLink: '',
      linkCopied: false,
    }
  },
  created(){
    this.shareableLink = this.getDomain('task-form') + this.config.token
    this.enableShareableLink = this.config.enabled
  },
  methods: {
    enableToggle(){
      this.btnLoading = true
      this.$emit('enableShareableLink', !this.enableShareableLink)

      Axios()
        .post(
          'config-form-answers/enable/toggle/?company_slug=' + 
          this.$route.params.companySlug + 
          '&project_slug=' + 
          this.$route.params.projectSlug
        )
        .then((response) => {
          this.btnLoading = false
        })
    },

    updateShareableLink(){
      this.enableShareableLinkLoading = true
      this.linkCopied = false
      this.config.shareableLink = ''
      this.loading = true

      Axios()
        .post(
          'config-form-answers/token/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
        )
        .then((response) => {
          this.token = response.data.data.token
          this.shareableLink = this.getDomain('task-form') + this.token
          this.enableShareableLinkLoading = false
          this.loading = false
        })
        .catch((e) => {
          this.enableShareableLinkLoading = false
        })
    },

    onCopy() {

      this.$copyText(this.tokenShareableBoard)
      this.linkCopied = false
      
    },
  }
}
</script>

<template>
<b-card :header="$t('Shareable Link')">
  <div class="d-flex justify-content-between">
    <b-form-checkbox
      v-model="enableShareableLink"
      @change="enableToggle">
        {{ $t('Enable Form2Task for this project') }}
    </b-form-checkbox>

    <div class="d-flex">
      <router-link
        v-show="enableShareableLink"
        :to="{ name: 'projects.addons.task-form.public',
        params: { token: config.token } }" target="_blank" class="small font-weight-bold text-primary">
          {{ $t('See my Form2Task') }}
      </router-link>
      <router-link v-if="goto === 'answers'" :to="{ name: 'projects.addons.task-form' }" class="small font-weight-bold ml-3">
        {{ $t('Go to Answers') }}
      </router-link>
      <router-link v-if="goto === 'settings'" :to="{ name: 'projects.addons.task-form.settings' }" class="small font-weight-bold ml-3">
        {{ $t('Go to Settings') }}
      </router-link>
    </div>
  </div>

  <b-input-group v-show="enableShareableLink" class="mt-2">
    <b-input v-model="shareableLink" 
      :readonly="true" 
      autocomplete="off"></b-input>
    <b-input-group-append>
      <b-button 
      v-clipboard:copy="shareableLink"
      v-clipboard:success="onCopy"
      variant="outline-secondary" >
        <font-awesome-icon :icon="['far', 'copy']" />
      </b-button>
    </b-input-group-append>
  </b-input-group>

  <a
  v-show="enableShareableLink"
  href="javascript:;"
  class="txt-68748F fw-500 lh-15-px tx-10-px tx-uppercase"
  @click="updateShareableLink">
  {{ $t('Update Link') }}
  </a>
</b-card>
</template>
