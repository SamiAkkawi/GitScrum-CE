<script>
export default {
  props: {
    page: {
      type: Number,
      required: true,
      default: 0,
    },
    totalPages: {
      type: Number,
      required: true,
      default: 0,
    },
    totalRows: {
      type: Number,
      required: true,
      default: 0,
    },
    perPage: {
      type: Number,
      required: true,
      default: 0,
    },
  },
  data() {
    return {}
  },
  methods: {
    change(value) {
      this.$emit('change', value)
    },
  }
}
</script>

<template>
<div v-if="totalPages > 1" class="d-flex justify-content-center mt-4">
  <b-pagination
    v-model="page"
    hide-goto-end-buttons
    class="paginator"
    :total-rows="totalRows"
    :per-page="perPage"
    @change="change">
    <template slot="prev-text">
      <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
    </template>
    <template slot="next-text">
      <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
    </template>
  </b-pagination>
</div>
</template>
