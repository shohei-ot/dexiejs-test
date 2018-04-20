<template>
  <tr class="table-row">
    <td v-for="(col, i) in cols" :key="i">{{ row[col] }}</td>
    <slot name="suffix-cell"></slot>
  </tr>
</template>

<script>
export default {
  props: {
    cols: {
      type: Array,
      reuqired: true
    },
    row: {
      type: Object,
      required: true
    }
  }
};
</script>

<style lang="scss" scoped>
$border-style: 1px solid #aaa;
.table-row {
  td {
    border-right: $border-style;
    border-bottom: $border-style;
    padding: 0 0.4rem;
  }
}
</style>