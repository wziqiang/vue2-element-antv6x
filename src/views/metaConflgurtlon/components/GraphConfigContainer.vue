<template>
  <div id="header" class="fx-s-center">
    <el-tooltip content="全屏" placement="top">
      <span
        class="iconfont header-iconfont"
        @click="
          () => {
            this.$emit('full', true);
          }
        "
        >&#xec13;</span
      >
    </el-tooltip>
    <el-tooltip content="退出全屏" placement="top">
      <span
        class="iconfont header-iconfont"
        @click="
          () => {
            this.$emit('full', false);
          }
        "
        >&#xeb11;</span
      >
    </el-tooltip>
    <el-tooltip content="新增列表" placement="top">
      <span class="iconfont header-iconfont" style="font-size: 14px;" @click="add">&#xe623;</span>
    </el-tooltip>
    <el-tooltip content="保存" placement="top">
      <span class="iconfont header-iconfont" @click="save">&#xe65b;</span>
    </el-tooltip>
  </div>
</template>
  
  <script>
import { Graph, DataUri } from "@antv/x6";
export default {
  name: "GraphConfigContainer",
  props: {
    graph: Graph,
  },
  methods: {
    verify() {
      return new Promise((resolve, reject) => {});
    },
    add(){
        this.$prompt("请输入名称", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(({ value }) => {
          this.$message({
            type: "success",
            message: "你的新增名称是: " + value,
          });
          this.$emit("add", value);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "取消输入",
          });
        });
    },
    // 保存
    save() {
      console.log(this.graph.toJSON());
      if (this.graph.toJSON().cells.length == 0) {
        this.$message.warning("请配置元素");
        return;
      }
      this.$emit("save");
    },
  },
};
</script>
  
  <style scoped>
#header {
  display: flex;
  border-top: 1px solid #dfe3e8;
  /* padding-top: 5px; */
  margin: 0;
  height: 35px;
  width: 100%;
}
.header-iconfont {
  margin-left: 20px;
  cursor: pointer;
  color: #595959;
}
</style>
  