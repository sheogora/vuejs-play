<template>
  <div class="table-container">
    <div>
      <input type="text" class="form-control" placeholder="Search" v-model="search">
    </div>
    <table class="table">
      <thead>
        <template v-for="column in columns">
          <th v-bind:style="getWidth(column.width)">
            <a class="alert-link" v-on:click="sortColumn(column)">{{column.title}}</a>
          </th>
        </template>
        <th> </th>
      </thead>
      <tbody>
        <tr v-for="(row, i) in display">
          <template>
            <td v-for="column in columns">
              <input type="text" v-model="row[column.name]" v-if="column.editable && isEdit(row)"/>
              <span v-else>{{ row[column.name] }}</span>
            </td>
          </template>
          <td>
            <button type="button" v-if="!isEdit(row)" v-on:click="toggleEdit(row)" class="btn btn-outline-primary">Edit</button>
            <button type="button" v-if="isEdit(row)" v-on:click="updateRow(row)" class="btn btn-outline-primary">Save</button>
            <button type="button" v-on:click="deleteRow(row)" :id="row.id" class="btn btn-outline-danger">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="v-paginator">
      <button class="btn btn-outline-primary" v-on:click="pagination('previous')">Previous</button>
      <span>Page {{current}} of {{end}}</span>
      <button class="btn btn-outline-primary" v-on:click="pagination('next')">Next</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'search-table',
  props: {
    apiUrl: {
      type: String,
      required: true
    },
    rows: {
      type: Number,
      default: 5,
      required: false
    },
    columns: {
      type: Array,
      required: true
    }
  },
  watch: {
    'search': function(val){
      let filtered = [];
      let temp = this.dataList;
      const that = this;
      if (val.length > 0) {
        let searchResult = temp.filter(function (el) {
          var searchable = "";
          for (var i = 0; i < that.columns.length; i++) {
            var column = that.columns[i];
            if(column.searchable){
              searchable = searchable.concat(el[column.name]);
              searchable = searchable.concat(',');
            }
          }

          if(searchable.toLowerCase().indexOf(val.toLowerCase()) > -1) {
            filtered.push(el);
          }
        });
        this.list = filtered;
      }
      else {
        this.list = this.dataList;
      }

      this.current = 1;
      this.pagination();
      this.$forceUpdate();
    }
  },
  data() {
    return {
      list: [],
      dataList: [],
      edit: [],
      search: "",
      sort: {},
      start: 1,
      end: 1,
      current: 1,
      display: []
    }
  },
  mounted() {
    this.loadData();
  },
  methods: {
    initialize: function() {
      this.dataList = this.list;
      this.sortColumn(this.columns[0]);
      this.end = Math.ceil(this.list.length / this.rows);
      this.pagination();
    },
    getWidth: function(style) {
      return {width: style};
    },
    isEdit: function(row) {
      var index = this.list.indexOf(row);
      var id = this.list[index].id;
      return this.edit[id];
    },
    toggleEdit: function(row){
      var index = this.list.indexOf(row);
      var id = this.list[index].id;
      this.edit[id] = this.edit[id] ? !this.edit[id] : true;
      this.$forceUpdate();
    },
    loadData: function() {
      var that = this;
      axios.get(this.apiUrl)
      .then(response => {
        that.list = response.data;
        that.initialize();
      })
      .catch(error => {
        console.log("error " + error);
      });
    },
    deleteRow: function(row) {
      axios.delete(this.apiUrl + row.id)
      .then(response => {
        var index = this.list.indexOf(row);
        this.list.splice(index, 1);
        this.pagination();
      })
      .catch(error => {
        console.log("error " + error);
      });
    },
    updateRow(row) {
      axios.put(this.apiUrl + row.id, row)
      .then(response => {
        this.toggleEdit(row);
      })
      .catch(error => {
        console.log("error " + error);
      });
    },
    sortColumn: function(column) {
      if (this.sort.column === column.name){
        this.sort.order = -this.sort.order;
      }
      else {
        this.sort.column = column.name;
        this.sort.order = 1;
      }

      const that = this;
      this.list.sort(function (a,b) {
        let i = that.sort.column;
        var result = (a[i] < b[i]) ? -1 : (a[i] > b[i]) ? 1 : 0;
        return result * that.sort.order;
      });

      this.pagination();
      this.$forceUpdate();
    },
    pagination: function(action) {
      var newList = [];
      if (action === 'next') {
        this.current = this.current+1;
      }
      else if (action === 'previous') {
        this.current = this.current-1;
      }
      var current = this.current*this.rows;
      for (let i = current-this.rows; i < current; i++) {
        if(this.list[i]){
          newList.push(this.list[i]);
        }
      }
      this.display = newList;
    }
  }
}
</script>

<style lang="scss">

</style>
