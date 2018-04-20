<template>
  <thead class="table-head">
    <tr class="table-head">
      <th v-for="(col, i) in localCols" :key="i">
        <span>{{col.name}}</span>
        <select v-model="col.order" @change="orderBy(col.name, col.order === 'asc' ? 'desc' : 'asc')">
          <option value="">-</option>
          <option value="asc">▲</option>
          <option value="desc">▼</option>
        </select>
      </th>
      <slot name="suffix-cell"></slot>
    </tr>
  </thead>
</template>

<script>
export default {
  props: {
    cols: {
      type: Array,
      reuqired: true
    }
  },
  data() {
    return {
      sort: "",
      order: "asc",
      localCols: []
    };
  },
  created() {
    this.initLocalCols();
  },
  methods: {
    initLocalCols() {
      console.log("initLocalCols");
      this.localCols = this.cols.map(colName => {
        const col = {
          name: colName
        };
        col.order = "";
        return col;
      });
    },
    orderBy(key, order) {
      order = order.toLowerCase();
      this.setSortKey(key);
      this.setOrderBy(order);
    },
    setSortKey(key) {
      if (!this.cols.includes(key)) {
        return false;
      }
      this.$data.sort = key;
    },
    setOrderBy(order) {
      order = order.toLowerCase();
      if (!["asc", "desc"].includes(order)) {
        return false;
      }
      this.$data.order = order;
    },
    getOrderBy() {
      return {
        sort: this.sort,
        order: this.order
      };
    },
    emitOrder() {
      if (this._emitOrderTimer) {
        clearTimeout(this._emitOrderTimer);
        delete this._emitOrderTimer;
      }
      this._refreshLocalColSortParams();
      this._emitOrderTimer = setTimeout(() => {
        const d = { sort: this.sort, order: this.order };
        console.log("emitOrder", d);
        this.$emit("order", d);
      }, 0);
    },
    _refreshLocalColSortParams() {
      const sort = this.sort;
      this.localCols.forEach(col => {
        if (col.name !== sort) {
          col.order = "";
        }
      });
    }
  },
  watch: {
    cols: {
      handler() {
        this.initLocalCols();
      },
      deep: true
    },
    sort() {
      this.emitOrder();
    },
    order() {
      this.emitOrder();
    }
  }
};
</script>

<style lang="scss" scoped>
$border-style: 1px solid #aaa;
.table-head {
  th {
    border-right: $border-style;
    border-bottom: $border-style;
    padding: 0 0.4rem;
  }
}
</style>