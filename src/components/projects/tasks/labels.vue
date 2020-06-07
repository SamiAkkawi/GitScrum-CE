<script>
import hexToRgba from 'hex-to-rgba'

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
    backgroundColor(hexColor) {
      return hexToRgba(hexColor, 0.2)
    },
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
      :style="'color:' + label.color + ';background:' + backgroundColor(label.color)" 
      v-text="label.title">
    </span>
  </div>
</template>
