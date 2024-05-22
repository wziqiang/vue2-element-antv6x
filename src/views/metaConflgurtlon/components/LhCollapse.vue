<template>
  <div class="Collapse_box">
    <div class="title" @click="show = !show">
      <i v-show="show" class="el-icon-remove-outline"></i
      ><i v-show="!show" class="el-icon-circle-plus-outline"></i>
      <span style="margin-left: 2px">{{ title }}</span>
    </div>
    <div v-show="show" class="content">
      <slot></slot>
    </div>
  </div>
</template>
  
  <script>
export default {
  props: {
    title: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      show: true,
    };
  },
};
</script>
  
  <style  lang="scss" scoped>
.Collapse_box {
  width: 100%;
  height: auto;
  .title {
    width: 100%;
    height: 32px;
    padding: 0 5px 0 8px;
    color: #666;
    background: #00000008;
    font-weight: 700;
    font-size: 14px;
    line-height: 32px;
    cursor: pointer;
    &:hover {
      color: #444;
    }
  }
  .content {
    margin: 10px 0;
  }
}
</style>