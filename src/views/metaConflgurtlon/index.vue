<template>
  <div class="dashboard-container" :class="visible ? 'full-screen' : ''">
    <graphconfigcontainer
      :graph="this.graph"
      @full="handleFull"
      @save="save"
      @add="add"
    ></graphconfigcontainer>
    <div class="antvBox">
      <div class="box-left">
        <lhCollapse title="列表">
          <contenttable ref="contenttable"   :graph="this.graph" @setgraph="setgraph" />
        </lhCollapse>
        <lhCollapse title="元件元素">
          <div class="menu-list">
            <div
              v-for="item in moduleList"
              :key="item.id"
              draggable="true"
              @dragend="handleDragEnd($event, item)"
              @ondragover="dragOverHandler($event)"
              @ondrop="dragOverHandler($event)"
            >
              <img :src="item.image" alt="" />
              <p>{{ item.name }}</p>
            </div>
          </div>
        </lhCollapse>
      </div>

      <div
        class="canvas-card"
        :style="
          configVisibe
            ? 'width: calc(100% - 265px)'
            : 'width:calc(100% - 510px)'
        "
      >
        <div id="container" />
      </div>
      <div class="box-right">
        <span
          class="iconfont collpase-icon"
          @click="configVisibe = !configVisibe"
        >
          {{ configVisibe ? "&#xe627;" : "&#xe609;" }}</span
        >
        <nodeconfigcontainer
          style="width: 250px"
          v-show="!configVisibe"
          :node="curNode"
          @updateData="handleNode"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { Graph } from "@antv/x6";
import { Snapline } from "@antv/x6-plugin-snapline"; //对齐线
import lhCollapse from "@/views/metaConflgurtlon/components/LhCollapse";
import lhproperties from "@/views/metaConflgurtlon/components/LhProperties";
import contenttable from "@/views/metaConflgurtlon/components/contentTable";
import nodeconfigcontainer from "@/views/metaConflgurtlon/components/NodeConfigContainer";
import graphconfigcontainer from "@/views/metaConflgurtlon//components/GraphConfigContainer";
export default {
  name: "antvX6",
  computed: {
    ...mapGetters(["sidebar"]),
  },
  components: {
    lhproperties,
    nodeconfigcontainer,
    lhCollapse,
    contenttable,
    graphconfigcontainer,
  },
  data() {
    return {
      visible: false, //是否全屏
      configVisibe: false, //收缩配置面板
      curNode: null,
      curSelectNode: null,
      graph: null,
      moduleList: [
        {
          id: 1,
          name: "分布式电源",
          image: require("@/assets/svg/fbsdy.svg"),
          data: {
            Powersource: "电源类型",
            Activepowerrange: "有功功率范围",
            Reactivepowerrange: "无功功率范围",
          },
        },
        {
          id: 8,
          name: "储能",
          image: require("@/assets/svg/cn.svg"),
        },
        {
          id: 2,
          name: "充电桩",
          image: require("@/assets/svg/cdz.svg"),
        },
        {
          id: 4,
          name: "负荷",
          image: require("@/assets/svg/fh.svg"),
        },
        {
          id: 5,
          name: "变压器",
          image: require("@/assets/svg/byq.svg"),
        },
        {
          id: 6,
          name: "开关",
          image: require("@/assets/svg/kg.svg"),
        },
      ],
    };
  },
  mounted() {
    this.initGraph();
  },
  methods: {
    setgraph(graph) {
      this.graph.fromJSON(graph);
    },
    add(name) {
      this.graph.clearCells();
      const graphArr = localStorage.getItem("graphArr");
      if (graphArr) {
        let Arr = JSON.parse(graphArr);
        Arr.unshift({ name, graph: this.graph.toJSON(), id: new Date() });
        localStorage.setItem("graphArr", JSON.stringify(Arr));
      } else {
        let Arr = [{ name, graph: this.graph.toJSON(), id: new Date() }];
        localStorage.setItem("graphArr", JSON.stringify(Arr));
      }
      this.$refs.contenttable.getData();
    },
    //保存画布，并提交
    save() {
      console.log(this.graph.toJSON(), "graph");
      console.log(this.graph.getNodes(), "node");
      console.log(this.$refs.contenttable.currentRow);
      const id = this.$refs.contenttable.currentRow.id;
      let arr = JSON.parse(localStorage.getItem("graphArr"));
      console.log(arr);
      arr.map((e) => {
        if (e.id == id) {
          e.graph = this.graph.toJSON();
        }
      });
      localStorage.setItem("graphArr", JSON.stringify(arr));
      this.$refs.contenttable.getData();
    },
    // 是否全屏
    handleFull(value) {
      this.visible = value;
    },
    // 更新node的数据
    handleNode(data) {
      console.log("更新的数据", data);
      this.curNode = data;
    },
    initGraph() {
      const container = document.getElementById("container");
      this.graph = new Graph({
        container: container, // 画布容器
        autoResize: true, //画布自适应
        width: container.offsetWidth, // 画布宽
        height: container.offsetHeight, // 画布高
        background: false, // 背景（透明）
        // 配置连线规则
        connecting: {
          snap: true, // 自动吸附
          allowBlank: false, // 是否允许连接到画布空白位置的点
          allowMulti: true, // 是否允许在相同的起始节点和终止之间创建多条边
          allowLoop: true, // 是否允许创建循环连线，即边的起始节点和终止节点为同一节点
          highlight: true, // 拖动边时，是否高亮显示所有可用的节点
          highlighting: {
            magnetAdsorbed: {
              name: "stroke",
              args: {
                attrs: {
                  fill: "#5F95FF",
                  stroke: "#5F95FF",
                },
              },
            },
          },
          router: {
            // 对路径添加额外的点
            name: "orth",
          },
          connector: {
            // 边渲染到画布后的样式
            name: "rounded",
            args: {
              radius: 8,
            },
          },
        },
        panning: {
          enabled: false,
        },
        mousewheel: {
          enabled: true, // 支持滚动放大缩小
          zoomAtMousePosition: true,
          modifiers: "ctrl",
          minScale: 0.5,
          maxScale: 3,
        },
        grid: {
          type: "dot",
          size: 20, // 网格大小 10px
          visible: true, // 渲染网格背景
          args: {
            color: "#a0a0a0", // 网格线/点颜色
            thickness: 2, // 网格线宽度/网格点大小
          },
        },
        //限制节点拖拽动到画布之外
        translating: {
          restrict: true,
        },
      });
      // 对齐线
      this.graph.use(new Snapline({ enabled: true, className: "my-snapline" }));
      this.nodeAddEvent();
    },
    //添加节点到画布
    addHandleNode(x, y, id, image, name) {
      this.graph.addNode({
        id: id,
        shape: "image", // 指定使用何种图形，默认值为 'rect'
        x: x,
        y: y,
        width: 60,
        height: 60,
        imageUrl: image,
        // 增加自定义字段
        data: {
          name: "cc",
          customField: "customValue",
        },
        attrs: {
          body: {
            strokeWidth: 1,
            stroke: "#000000",
            fill: "#ffffff",
            rx: 10,
            ry: 10,
          },
          label: {
            textWrap: {
              width: 90,
              text: name,
            },
            fill: "black",
            fontSize: 12,
            refX: 0.5,
            refY: "100%",
            refY2: 4,
            textAnchor: "middle",
            textVerticalAnchor: "top",
          },
        },
        ports: {
          groups: {
            top: {
              position: "top",
              attrs: {
                circle: {
                  r: 4,
                  magnet: true,
                  stroke: "#5F95FF",
                  strokeWidth: 1,
                  fill: "#fff",
                  style: {
                    visibility: "hidden",
                  },
                },
              },
            },
            right: {
              position: "right",
              attrs: {
                circle: {
                  r: 4,
                  magnet: true,
                  stroke: "#5F95FF",
                  strokeWidth: 1,
                  fill: "#fff",
                  style: {
                    visibility: "hidden",
                  },
                },
              },
            },
            bottom: {
              position: "bottom",
              attrs: {
                circle: {
                  r: 4,
                  magnet: true,
                  stroke: "#5F95FF",
                  strokeWidth: 1,
                  fill: "#fff",
                  style: {
                    visibility: "hidden",
                  },
                },
              },
            },
            left: {
              position: "left",
              attrs: {
                circle: {
                  r: 4,
                  magnet: true,
                  stroke: "#5F95FF",
                  strokeWidth: 1,
                  fill: "#fff",
                  style: {
                    visibility: "hidden",
                  },
                },
              },
            },
          },
          //连接点
          items: [
            {
              group: "top",
            },
            {
              group: "right",
            },
            {
              group: "bottom",
            },
            {
              group: "left",
            },
          ],
        },
        zIndex: 10,
      });
    },
    dragOverHandler(e) {
      console.log("**************", e);
      e.preventDefault(); //解决触发拖拽事件即打开浏览器新标签页问题
    },
    // 拖动后松开鼠标触发事件
    handleDragEnd(e, item) {
      console.log("++++++++++++++++", e);
      e.preventDefault(); //解决触发拖拽事件即打开浏览器新标签页问题
      console.log(e, item); // 可以获取到最后拖动后松开鼠标时的坐标和拖动的节点相关信息
      const opened = this.sidebar.opened;
      const pageX = opened ? 440 : 280;
      const pageY = opened ? 100 : 120;
      this.addHandleNode(
        e.pageX - pageX,
        e.pageY - pageY,
        new Date().getTime(),
        item.image,
        item.name
      );
    },
    nodeAddEvent() {
      const { graph } = this;
      const container = document.getElementById("container");
      const changePortsVisible = (visible) => {
        const ports = container.querySelectorAll(".x6-port-body");
        for (let i = 0, len = ports.length; i < len; i = i + 1) {
          ports[i].style.visibility = visible ? "visible" : "hidden";
        }
      };
      this.graph.on("node:mouseenter", () => {
        changePortsVisible(true);
      });
      this.graph.on("node:mouseleave", () => {
        changePortsVisible(false);
      });
      //点击连接线事件
      this.graph.on("edge:click", ({ e, x, y, cell, edge, view }) => {
        console.log(edge);
      });
      // 连线绑定悬浮事件
      this.graph.on("cell:mouseenter", ({ cell }) => {
        if (cell.shape == "edge") {
          cell.addTools([
            {
              name: "button-remove",
              args: {
                x: "100%",
                y: 0,
                offset: {
                  x: 0,
                  y: 0,
                },
              },
            },
          ]);
          cell.setAttrs({
            line: {
              stroke: "#409EFF",
            },
          });
          cell.zIndex = 99; // 保证当前悬停的线在最上层，不会被遮挡
        }
      });
      this.graph.on("cell:mouseleave", ({ cell }) => {
        if (cell.shape === "edge") {
          // 连接线增加自定义字段
          cell.data = {
            name: "测试",
          };
          cell.removeTools();
          cell.setAttrs({
            line: {
              stroke: "black",
            }, //连接线颜色
          });
          cell.zIndex = 1; // 保证未悬停的线在下层，不会遮挡悬停的线
        }
      });
      // 监听画布添加节点动作
      this.graph.on("node:added", ({ node, index, options }) => {
        this.curNode = node;
        // todo 监听到画布添加了node之后，调后端接口创建一个接口
        // console.log('node:added')
        // console.log(node)
        // console.log(node.attrs.text.fontSize)
        // node.setLabel('xwtest') // 修改label的值
        // console.log('监听到拖入一个控件id：' + node.id)
        // console.log(this.graph.getNodes())
      });
      // 节点绑定点击事件
      this.graph.on("node:click", ({ e, x, y, node, view }) => {
        this.curNode = node;
        console.log("点击！！！", node);
        // 判断是否有选中过节点
        if (this.curSelectNode) {
          // 移除选中状态
          this.curSelectNode.removeTools();
          // 判断两次选中节点是否相同
          if (this.curSelectNode !== node) {
            node.addTools([
              {
                name: "boundary",
                args: {
                  attrs: {
                    fill: "#16B8AA",
                    stroke: "#2F80EB",
                    strokeWidth: 1,
                    fillOpacity: 0.1,
                  },
                },
              },
              {
                name: "button-remove",
                args: {
                  x: "100%",
                  y: 0,
                  offset: {
                    x: 0,
                    y: 0,
                  },
                },
              },
            ]);
            this.curSelectNode = node;
          } else {
            this.curSelectNode = null;
          }
        } else {
          this.curSelectNode = node;
          node.addTools([
            {
              name: "boundary",
              args: {
                attrs: {
                  fill: "#16B8AA",
                  stroke: "#2F80EB",
                  strokeWidth: 1,
                  fillOpacity: 0.1,
                },
              },
            },
            {
              name: "button-remove",
              args: {
                x: "100%",
                y: 0,
                offset: {
                  x: 0,
                  y: 0,
                },
              },
            },
          ]);
        }
      });
    },
  },
};
</script>
<style lang="scss" scoped>
.dashboard-container {
  background: #fff;
  width: 100%;
  height: 98%;
  .antvBox {
    border: 1px dashed #ccc;
    // border-right: 0px;
    // border-left: 0px;
    display: flex;
    width: 100%;
    height: 100%;
    color: black;
    // margin-top: 15px;
    .box-left {
      width: 250px;
      .menu-list {
        height: 100%;
        width: 100%;
        padding: 0 10px;
        display: flex;
        justify-content: center;
        align-content: center;
        flex-wrap: wrap;
        > div {
          margin-bottom: 5px;
          border-radius: 5px;
          padding: 0 5px;
          cursor: pointer;
          color: black;
          width: 90px;
          display: flex;
          flex-wrap: wrap;
          justify-content: center;
          img {
            height: 30px;
            width: 30px;
          }

          P {
            width: 90px;
            text-align: center;
          }
        }
      }
    }
    .box-right {
      position: relative;
    }
    .canvas-card {
      height: 100%;
      > div {
        width: 100%;
        height: 100%;
        border: 1px dashed #ccc;
        // border-right: 0px;
        // border-left: 0px;
      }
    }
  }
  .collpase-icon {
    position: absolute;
    top: 25px;
    left: -9px;
    color: #aaa;
    background-color: #fff;
    z-index: 3;
    cursor: pointer;
  }
}
.full-screen {
  position: fixed;
  overflow: auto;
  z-index: 1;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: #fff;
  border: 0;
  border-radius: 0px;
  padding: 0;
  margin: 0;
  height: 100%;
}
</style>