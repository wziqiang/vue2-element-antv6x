<template>
  <el-tabs v-model="activeName" type="card">
    <el-tab-pane label="属性" name="first">
      <el-form ref="form" :model="form" label-width="110px">
        <el-form-item label="电源类型">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="有功功率范围">
            <el-input v-model="form.Activepowerrange"></el-input>
        </el-form-item>
        <el-form-item label="无功功率范围">
            <el-input v-model="form.Reactivepowerrange"></el-input>
        </el-form-item>
        <!-- <el-form-item label="特殊资源">
          <el-radio-group v-model="form.resource">
            <el-radio label="线上品牌商赞助"></el-radio>
            <el-radio label="线下场地免费"></el-radio>
          </el-radio-group>
        </el-form-item> -->
      </el-form>
    </el-tab-pane>
  </el-tabs>
</template>
<script>
import {Node} from '@antv/x6'
export default {
  name: "NodeConfigContainer",
  props: {
    node: Node,
  },
  components: {},
  data() {
    return {
      activeName: "first",
      curNode: null,
      curLabel: "",
      form:{}
    };
  },
  created(){
    console.log(this.node);
  },
  methods: {

  },
  watch: {
    node(newValue) {
      this.curNode = newValue;
    },
    curNode(newValue) {
      this.$emit("updateData", newValue);
    },
  },
};
</script>
  
  <style scoped>
.color-container {
  margin-top: 8px;
  width: 24px;
  height: 24px;
  padding: 4px;
  border: 1px solid #dfe3e8;
  border-radius: 2px;
}
.edit-color {
  cursor: pointer;
  height: 100%;
}
</style>
  