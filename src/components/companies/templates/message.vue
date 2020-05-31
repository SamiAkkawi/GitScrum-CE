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
  <b-card>
    <b-card-text>
        {{ $t('You can create templates for your company and apply them to projects or apply automatically whenever a project is created.') }} 
        <b-link>{{ $t('Go to our Marketplace and get inspired by public templates') }}</b-link>  
    </b-card-text>
    </b-card>
</template>
