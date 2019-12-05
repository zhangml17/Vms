<template>
  <div class="graph-wrapper">
    <div class="img-drag-wrapper">
      <!-- img 为拖动源 -->
      <img src='../../../public/images/gear.png' ref="img_drag" />
    </div>

    <!-- <div class="drag-preview" ref="drag_preview" v-show="isDragged"></div> -->
    <!-- 整个graph容器 -->
    <div class="graph-container" ref="graph_container" :style="{backgroundImage:`url(${backgroundImage})`}">

    </div>
  </div>
</template>

<script>

// import { mxGraph, mxClient, mxUtils, mxGraphHandler, mxEdgeHandler, mxDragSource, mxCell, mxGuide} from 'mxgraph/javascript/mxClient'
import mxgraph from '../../utils/index'
const  {mxGraph, mxClient, mxUtils, mxGraphHandler, mxEdgeHandler,
        mxDragSource, mxCell, mxGuide, mxEvent, mxGeometry } = mxgraph

export default {
  data() {
    return {
      graph: null,
      isBrowserSupported: true,
      isDragged: false,
      errorMessage: 'Browser is not supported!',
      backgroundImage: require('../../../public/images/grid.gif'),
      dragImage:require('../../../public/images/gear.png')
    }
  },
  methods: {
    graphF(evt) {
      var x = mxEvent.getClientX(evt);
      var y = mxEvent.getClientY(evt);
      var elt = document.elementFromPoint(x, y);

      if (mxUtils.isAncestorNode(this.graph.container, elt))
      {
        return this.graph;
      }
        return null;
    },
    // 在指定位置插入cell（拖动完成后执行的方法）
    funct(graph, evt, target, x, y) {
      var cell = new mxCell('', new mxGeometry(0, 0, 48, 48), `shape=image;image=${this.dragImage}`);
      cell.vertex = true;
      var cells = this.graph.importCells([cell], x, y, target);

      if (cells != null && cells.length > 0)
      {
        this.graph.scrollCellToVisible(cells[0]);
        this.graph.setSelectionCells(cells);
      }
    },
    isGuidesEnabled() {
      return this.graph.graphHandler.guidesEnabled;
    }
  },
  created() {
    // 检查浏览器是否支持
    if (!mxClient.isBrowserSupported())
    {
      // 不支持，提示信息
      mxUtils.error('Browser is not supported!', 200, false);
      this.isBrowserSupported = false
    }
  },
  mounted() {
    if(this.isBrowserSupported) {
      // 拖动时显示导航线
      mxGraphHandler.prototype.guidesEnabled = true;
      // 当按下Alt键时不显示导航线
      mxGuide.prototype.isEnabledForEvent = function(evt)
      {
        return !mxEvent.isAltDown(evt);
      };
      mxEdgeHandler.prototype.snapToTerminals = true;
      // 创建graph
      this.graph = new mxGraph(this.$refs.graph_container)
      // 在IE中禁用内置DnD
      if (mxClient.IS_IE)
      {
        mxEvent.addListener(this.$refs.img_drag, 'dragstart', function(evt)
        {
          evt.returnValue = false;
        });
      }

      // 创建拖动预览元素.
      var dragElt = document.createElement('div');
      dragElt.style.border = 'dashed black 1px';
      dragElt.style.width = '48px';
      dragElt.style.height = '48px';

      // 构建mxDragSource实例
      var ds = mxUtils.makeDraggable(this.$refs.img_drag, this.graphF, this.funct, this.$refs.drag_preview, 0, 0, this.graph.autoscroll, true);

      ds.isGuidesEnabled = this.isGuidesEnabled
      // 在graph外部显示原始的拖动源图标样子
      ds.createDragElement = mxDragSource.prototype.createDragElement;
    }
  }
}
</script>

<style lang="scss" scoped>

.graph-wrapper {
  .img-drag-wrapper {
    background: #666565;
    img {
      width: 48px;
      height: 48px;
    }
  }

  .graph-container {
    width: 100%;
    height:400px;
  }
}

</style>
