<script>
export default {
  props: {
    template: {
      type: String,
      required: true,
    },
  },
  data() {
    return {}
  },
}
</script>

<template>
  <b-card :title="$t('GitScrum Marketplace')">
    <b-card-text>
        {{ $t('You can create templates for your company to apply automatically whenever a project is created') }}.
        <b-link :to="{ name: 'marketplace.templates' }">{{ $t('Go to our Marketplace and get inspired by public templates') }}</b-link>.
    </b-card-text>
    </b-card>
</template>
