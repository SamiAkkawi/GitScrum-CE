<script>

export default {
  components: {},
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
  },
  methods: {
  },
}
</script>

<template>
  <div v-if="task.labels[0]">
    <p class="mb-0 font-weight-bold pr-2">{{ $t('Task Labels') }}</p>
    <span
    v-for="label in task.labels"
    :key="label.slug"
    :alt="label.title"
    :title="label.title"
    class="badge mr-2"
    :style="'border:1px solid ' + 
    label.color + 
    ';color: ' + 
    invertColor(label.color, true) + 
    ' !important ;background:' + 
    opacityColor(label.color, '0.6') + 
    ' !important'"
    v-text="label.title" />
  </div>
</template>
