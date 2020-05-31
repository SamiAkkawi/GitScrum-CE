<script>

export default {
  props: {
    templates: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      templateSelected: null,
    }
  },
  mounted(){
    this.templateSelected = this.templates[0]
    this.selected(this.templates[0]);
    this.$emit('selected', this.templates[0])
  },
  methods: {
    selected(template){
      this.templateSelected = template
      this.$emit('selected', template)
    }
  },
}
</script>

<template>
  <b-card>
    <b-card-title>{{ $t('Templates available in your company') }}</b-card-title>
    <b-card-text>
      <div
        v-for="template in templates"
        :key="template.id"
        class="col-md-12 overflow-auto card-template cursor-pointer mb-10-px p-10-px"
        :class="templateSelected && templateSelected.slug === template.slug ? 'selected' : ''"
        @click="selected(template)">
        <p class="fw-600 txt-12-px txt-3D4F9F lh-18-px mb-0" v-text="template.name"></p>
        <p class="txt-909CB8 m-0"> <span v-text="template.items.length"></span> {{ $t('stages') }} </p>
      </div>
    </b-card-text>
  </b-card>
</template>
