<script>
export default {
  props: {
    labels: {
      type: Array,
      required: false,
      default: null,
    },
    customClass: {
      type: String,
      required: false,
      default: null,
    },
    limit: {
      type: Number,
      default: 20,
    },
  },
  methods: {
  },
}
</script>

<template>
  <div v-if="labels[0]" class="task-labels mt-2">
    <span
      v-for="label in labels"
      :key="label.slug"
      :class="customClass"
      :alt="label.title"
      :title="label.title"
      class="badge badge-light mb-1"
      :style="'border:1px solid ' + label.color  + ';color: ' + invertColor(label.color, true) + ';background:' + opacityColor(label.color, '0.2') + ' !important'">
      <div v-if="limit">{{ label.title | truncate(limit) }}</div>
      <div v-if="!limit">{{ label.title }}</div>
      <span v-show="label.total">({{ label.total }})</span>
    </span>
  </div>
</template>
