<template>
  <div class="contenttable_box">
    <el-table
      ref="singleTable"
      :data="tableData"
      highlight-current-row
      @current-change="handleCurrentChange"
      style="width: 100%"
    >
      <el-table-column property="name" label="名称" align="center">
      </el-table-column>
      <el-table-column align="center" label="操作">
        <template slot-scope="scope">
          <el-button @click="deleteClick(scope.row)" type="text" size="small"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { Graph } from "@antv/x6";
export default {
  props: {
    graph: Graph,
  },
  data() {
    return {
      tableData: [],
      currentRow: null,
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      const graphArr = JSON.parse(localStorage.getItem("graphArr"));
      this.tableData = graphArr || [];
      if (this.currentRow) {
        const row = this.tableData.find((e) => e.id == this.currentRow.id);
        this.setCurrent(row);
      } else {
        this.$nextTick(() => {
          this.tableData[0] && this.setCurrent(this.tableData[0]);
        });
      }
    },
    deleteClick(row) {
      this.$confirm("确定删除嘛, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          // this.$message({
          //   type: 'info',
          //   message: '已取消删除'
          // });
        });
      console.log(row);
    },
    setCurrent(row) {
      this.$refs.singleTable.setCurrentRow(row);
    },
    handleCurrentChange(val) {
      console.log(val);
      this.currentRow = val;
      this.$emit("setgraph", val.graph);
    },
  },
};
</script>
<style lang="scss" scoped>
.contenttable_box {
  ::v-deep th {
    padding: 0;
  }
  ::v-deep td {
    padding: 0;
  }
}
</style>