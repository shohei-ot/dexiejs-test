<template>
  <div class="container">
    <div>
      <label>name: <input type="text" v-model="form.name" placeholder="名前"></label>
      <label>foo: <input type="text" v-model="form.foo" placeholder="toto"></label>
      <button @click="inesrtRecord" type="button" :disabled="!form.name.length || !form.foo.length">insert</button>
      <button @click="dropDB" style="float: right">DB初期化</button>
    </div>
    <hr>
    <table class="records-table">
      <TableHead :cols="cols" @order="handleOrder">
        <th slot="suffix-cell">
          <span>&nbsp;</span>
        </th>
      </TableHead>
      <tbody>
        <TableRow v-for="(row, i) in rows" v-if="row" :key="i" :cols="cols" :row="row">
          <td slot="suffix-cell">
            <button class="del-btn" @click="delRecord(row.id)">delete</button>
          </td>
        </TableRow>
      </tbody>
    </table>
  </div>
</template>

<script>
import Dexie from "dexie";

import TableHead from "./components/Atoms/table-head";
import TableRow from "./components/Atoms/table-row";

export default {
  components: {
    TableHead,
    TableRow
  },
  data() {
    return {
      form: {
        name: "",
        foo: ""
      },
      dbName: "MyDatabase",
      db: null,
      table: null,
      collection: null,

      cols: [],
      rows: [],
      sortKey: "id",
      order: "asc"
    };
  },
  mounted() {
    this.initDB();
  },
  methods: {
    async initDB() {
      const db = new Dexie(this.$data.dbName);
      window.db = db;
      this.$data.db = db;

      if (!db.tables.length) {
        db.version(1).stores({
          users: "++id,&name,foo"
        });
      }
      this.$data.table = db.users;
      this.$data.collection = db.users.toCollection();

      if (!await db.users.count()) {
        console.log("no users");
        db.users.add({ name: "Shohei Ota", foo: "bar" });
      }

      this.$data.cols = ["id", "name", "foo"];
      this.$nextTick(() => {
        this.collectionToRows();
      });
    },

    handleOrder(e) {
      if (this.table === null) {
        return false;
      }
      console.log("handleOrder", e);
      this.collectionToRows(e);
    },

    async inesrtRecord() {
      try {
        const newRecord = JSON.parse(JSON.stringify(this.$data.form));
        const keys = Object.keys(newRecord);
        await this.$data.table.add(newRecord);
        const obj = keys.reduce((accum, key) => {
          accum[key] = "";
          return accum;
        }, {});
        this.$data.form = obj;
      } catch (e) {
        alert(e.message || e);
        return false;
      }
      this.collectionToRows();
    },
    async delRecord(recordId) {
      await this.$data.table.delete(recordId);
      this.collectionToRows();
    },

    async collectionToRows(opts = {}) {
      console.log("collectionToRows");
      const rows = [];
      if (opts.sort && opts.order) {
        const { sort, order } = opts;
        let tmp = [];
        switch (order) {
          case "desc":
            tmp = await this.$data.collection.sortBy(sort);
          case "asc":
            tmp = await this.$data.collection.reverse().sortBy(sort);
        }
        for (const row of tmp) {
          rows.push(row);
        }
      } else {
        const tmp = await this.$data.collection.toArray();
        for (const row of tmp) {
          rows.push(row);
        }
      }
      this.$data.rows = rows;
    },

    async dropDB() {
      if (window.confirm("DBを初期化します")) {
        await this.$data.db.delete();
        await this.initDB();
      }
    }
  },
  watch: {
    table: {
      handler() {
        console.log("watch.table");
      },
      deep: true
    },
    collection: {
      handler() {
        console.log("watch.collection");
      },
      deep: true
    }
  }
};
</script>

<style lang="scss" scoped>
$border-style: 1px solid #aaa;
.container {
  .records-table {
    border-spacing: 0;
    border-top: $border-style;
    border-left: $border-style;
    .del-btn {
      cursor: pointer;
      border: 1px solid #800;
      font-size: 0.8rem;
      color: #800;
      background-color: #fff;
      border-radius: 2px;
      padding: 0.1rem 0.4rem;
      margin: 0.2rem;
      transition-duration: 80ms;
      &:hover {
        background-color: #fdd;
      }
    }
  }
}
</style>