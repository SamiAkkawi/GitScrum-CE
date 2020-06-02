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
  <b-card :header="$t('Templates available')">
    <b-card-text text-tag="div">
      <div
        v-for="template in templates"
        :key="template.id"
        class="col-md-12 card-template cursor-pointer p-2 mb-1"
        :class="templateSelected && templateSelected.slug === template.slug ? 'selected' : ''"
        @click="selected(template)">
        <p class="fw-600 mb-0" v-text="template.name"></p>
        <p class="mb-0 small"> <span v-text="template.items.length"></span> {{ $t('stages') }} </p>
      </div>
    </b-card-text>
  </b-card>
</template>
