<script>
export default {
  props: {
    visibility: {
      required: false,
      default: function() {
        return []
      },
    },
  },
}
</script>

<template>
<b-badge v-b-popover.hover.top=" visibility.title + ' ' + $t('Project')" variant="light">
  <span v-show="visibility.title">
    <font-awesome-icon v-if="!visibility.is_private" :icon="['fa', 'lock-open-alt']" style="font-size: 12px;" />
    <font-awesome-icon v-if="visibility.is_private" :icon="['fa', 'lock-alt']" style="font-size: 12px;" />
  </span>
 </b-badge>
</template>
