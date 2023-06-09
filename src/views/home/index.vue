<template>
  <div class="home">
    <Layout>
      <Header v-if="show">
        <!-- 导入 -->
        <div style="float: right">
          <!-- 预览 -->
          <history></history>
          <save></save>
          <lang></lang>
        </div>
      </Header>
      <Content style="display: flex; height: calc(100vh - 64px)">
        <div v-if="show" style="width: 380px; height: 100%; background: #fff; display: flex">
          <Menu
            :active-name="menuActive"
            accordion
            @on-select="(activeIndex) => (menuActive = activeIndex)"
            width="65px"
          >
            <MenuItem :name="1" class="menu-item">
              <Icon type="md-add" size="48" />
              <div>插入</div>
            </MenuItem>
            <MenuItem :name="2" class="menu-item">
              <Icon type="ios-brush" size="36" />
              <div>设计元素</div>
            </MenuItem>
            <MenuItem :name="3" class="menu-item">
              <Icon type="md-reorder" size="36" />
              <div>图层</div>
            </MenuItem>
            <MenuItem :name="4" class="menu-item">
              <Icon type="logo-freebsd-devil" size="36" />
              <div>贴图</div>
            </MenuItem>
            <MenuItem :name="5" class="menu-item">
              <Icon type="md-color-palette" size="36" />
              <div>背景设置</div>
            </MenuItem>
          </Menu>
          <div class="content">
            <!-- 生成模板 -->
            <div v-show="menuActive === 1" class="left-panel">
              <import-tmpl></import-tmpl>
            </div>
            <!-- 常用元素 -->
            <div v-show="menuActive === 2" class="left-panel">
              <tools></tools>
              <!-- <fontTmpl></fontTmpl> -->
            </div>
            <!-- 背景设置 -->
            <div v-show="menuActive === 3" class="left-panel">
              <layer></layer>
            </div>
            <div v-show="menuActive === 4" class="left-panel">
              <svgEl></svgEl>
            </div>
            <div v-show="menuActive === 5" class="left-panel">
              <bg-bar></bg-bar>
            </div>
          </div>
        </div>
        <!-- 画布区域 -->
        <div id="workspace" style="width: 100%; position: relative; background: #f1f1f1">
          <div class="canvas-box">
            <div class="inside-shadow"></div>
            <canvas id="canvas" :class="ruler ? 'design-stage-grid' : ''"></canvas>
            <dragMode></dragMode>
            <zoom></zoom>
            <mouseMenu></mouseMenu>
          </div>
        </div>
        <!-- 属性区域 380-->
        <div style="width: 530px; height: 100%; padding: 10px; overflow-y: auto; background: #fff">
          <div v-if="show" style="padding-top: 10px">
            <!-- 新增字体样式使用 -->
            <set-size></set-size>
            <group></group>
          </div>
          <attribute v-if="show"></attribute>
        </div>
      </Content>
    </Layout>
  </div>
</template>

<script>
import save from '@/components/save.vue';
import lang from '@/components/lang.vue';
import group from '@/components/group.vue';
import zoom from '@/components/zoom.vue';
import dragMode from '@/components/dragMode.vue';

// 左侧组件
import importTmpl from '@/components/importTmpl.vue';
import tools from '@/components/tools.vue';
import svgEl from '@/components/svgEl.vue';
import bgBar from '@/components/bgBar.vue';
import setSize from '@/components/setSize.vue';
// import replaceImg from '@/components/replaceImg.vue';
// import filters from '@/components/filters.vue';

// 右侧组件
import history from '@/components/history.vue';
import layer from '@/components/layer.vue';
import attribute from '@/components/attribute.vue';

// 右键菜单
import mouseMenu from '@/components/contextMenu/index.vue';

// 功能组件
import CanvasEventEmitter from '@/utils/event/notifier';
import { downFile } from '@/utils/utils';
import { fabric } from 'fabric';
import Editor from '@/core';

const event = new CanvasEventEmitter();
const canvas = {};
export default {
  name: 'HomeView',
  provide: {
    canvas,
    fabric,
    event,
  },
  data() {
    return {
      menuActive: 1,
      show: false,
      select: null,
      ruler: true,
    };
  },
  components: {
    setSize,
    tools,
    bgBar,
    layer,
    attribute,
    dragMode,
    save,
    lang,
    importTmpl,
    group,
    zoom,
    svgEl,
    history,
    mouseMenu,
  },
  watch: {
    ruler: {
      handler(value) {
        if (!this.canvas.ruler) return;
        if (value) {
          this.canvas.ruler.enable();
        } else {
          this.canvas.ruler.disable();
        }
      },
    },
  },
  mounted() {
    this.canvas = new fabric.Canvas('canvas', {
      fireRightClick: true, // 启用右键，button的数字为3
      stopContextMenu: true, // 禁止默认右键菜单
      controlsAboveOverlay: true, // 超出clipPath后仍然展示控制条
    });

    canvas.c = this.canvas;
    event.init(canvas.c);
    canvas.editor = new Editor(canvas.c);

    canvas.c.renderAll();

    this.show = true;
    this.$Spin.hide();
  },
  methods: {
    // 获取字体数据 新增字体样式使用
    getFontJson() {
      const activeObject = this.canvas.getActiveObject();
      if (activeObject) {
        const json = activeObject.toJSON(['id', 'gradientAngle', 'selectable', 'hasControls']);
        console.log(json);
        const fileStr = `data:text/json;charset=utf-8,${encodeURIComponent(
          JSON.stringify(json, null, '\t')
        )}`;
        downFile(fileStr, 'font.json');
        const dataUrl = activeObject.toDataURL();
        downFile(dataUrl, 'font.png');
      }
    },
  },
};
</script>
<style lang="less" scoped>
.logo {
  width: 30px;
  height: 30px;
  display: inline-block;
  margin-right: 10px;
  text-align: center;
  vertical-align: middle;
  .ivu-icon {
    vertical-align: super;
  }
}

// 属性面板样式
:deep(.attr-item) {
  position: relative;
  margin-bottom: 12px;
  height: 40px;
  padding: 0 10px;
  background: #f6f7f9;
  border: none;
  border-radius: 4px;
  display: flex;
  align-items: center;
  .ivu-tooltip {
    text-align: center;
    flex: 1;
  }
}

.ivu-menu-vertical .menu-item {
  text-align: center;
  padding: 10px 2px;
  box-sizing: border-box;
  font-size: 12px;

  & > i {
    margin: 0;
  }
}

:deep(.ivu-layout-header) {
  --height: 45px;
  padding: 0 10px;
  border-bottom: 1px solid #eef2f8;
  background: #fff;
  height: var(--height);
  line-height: var(--height);
}

.home,
.ivu-layout {
  height: 100vh;
}

.icon {
  display: block;
}

.canvas-box {
  position: relative;
}
// 画布内阴影
.inside-shadow {
  position: absolute;
  width: 100%;
  height: 100%;
  box-shadow: inset 0 0 9px 2px #0000001f;
  z-index: 2;
  pointer-events: none;
}

#canvas {
  width: 300px;
  height: 300px;
  margin: 0 auto;
}

#workspace {
  overflow: hidden;
}

.content {
  flex: 1;
  width: 220px;
  padding: 10px;
  padding-top: 0;
  height: 100%;
  overflow-y: auto;
}

.ivu-menu-light.ivu-menu-vertical .ivu-menu-item-active:not(.ivu-menu-submenu) {
  background: none;
}
// 标尺
.switch {
  margin-right: 10px;
}
// 网格背景
.design-stage-grid {
  --offsetX: 0px;
  --offsetY: 0px;
  --size: 16px;
  --color: #dedcdc;
  background-image: linear-gradient(
      45deg,
      var(--color) 25%,
      transparent 0,
      transparent 75%,
      var(--color) 0
    ),
    linear-gradient(45deg, var(--color) 25%, transparent 0, transparent 75%, var(--color) 0);
  background-position: var(--offsetX) var(--offsetY),
    calc(var(--size) + var(--offsetX)) calc(var(--size) + var(--offsetY));
  background-size: calc(var(--size) * 2) calc(var(--size) * 2);
}
</style>
