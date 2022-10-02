<template>
  <div>
    <h1>Classification list</h1>
    <el-table :data="items">
      <el-table-column prop="_id" label="ID" width="240"></el-table-column>
      <el-table-column prop="parent.name" label="Category parent"></el-table-column>
      <el-table-column prop="name" label="Classification name"></el-table-column>
      <el-table-column fixed="right" label="Operation" width="180">
        <template slot-scope="scope">
          <el-button type="primary" size="small" 
          @click="$router.push(`/categories/edit/${scope.row._id}`)">Edit</el-button>
          <el-button type="primary" size="small" 
          @click="remove(scope.row)">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: []
    };
  },
  methods: {
    async fetch() {
      const res = await this.$http.get("rest/categories");
      this.items = res.data;
    },
    async remove(row){
      this.$confirm(`Are you sure you want to delete the classification"${row.name}"`, 'Tips', {
          confirmButtonText: 'Determine',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(async () => {
          const res = await this.$http.delete(`rest/categories/${row._id}`)
          this.$message({
            type: 'success',
            message: 'Delete succeeded!'
          });
          this.fetch()
        })
    }
  },
  created() {
    this.fetch();
  }
};
</script>
