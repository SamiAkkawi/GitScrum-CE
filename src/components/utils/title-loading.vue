<script>
export default {
  props: {
    title: {
      type: String,
      required: false,
      default: '',
    },
    titleAlternative: {
      type: String,
      required: false,
      default: '',
    },
    subtitle: {
      type: String,
      required: false,
      default: null,
    },
    addclass: {
      type: String,
      required: false,
      default: '',
    },
    loading: {
      type: Boolean,
      required: false,
      default: false
    },
  },
}
</script>

<template>
  <div :class="addclass" class="titleLoading">
      <h1 class="">
        <span v-if="!titleAlternative" v-text="title"></span>
        <span v-if="!loading && titleAlternative" v-text="title"></span>
        <span v-if="loading" v-text="titleAlternative"></span>
        <b-spinner v-show="loading" :label="$t('Loading')" small class="ml-2"></b-spinner>
      </h1>
      <p v-if="subtitle" v-text="subtitle"></p>
  </div>
</template>
