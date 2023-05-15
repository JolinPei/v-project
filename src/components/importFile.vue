<template>
  <div style="display: inline-block">
    <Button type="primary" @click="insertImg">
      <Icon type="md-add-circle" />
      插入图片
    </Button>
    <Modal
      v-model="showModal"
      :title="$t('insertFile.modal_tittle')"
      @on-ok="insertSvgStr"
      @on-cancel="showModal = false"
    >
      <Input
        v-model="svgStr"
        show-word-limit
        type="textarea"
        :placeholder="$t('insertFile.insert_SVGStr_placeholder')"
      />
    </Modal>
  </div>
</template>

<script>
import { getImgStr, selectFiles } from '@/utils/utils';
import select from '@/mixins/select';
import { v4 as uuid } from 'uuid';

export default {
  name: 'ToolBar',
  mixins: [select],
  data() {
    return {
      showModal: false,
      svgStr: '',
    };
  },
  methods: {
    insertTypeHand(type) {
      this[type]();
    },
    // 插入图片
    insertImg() {
      selectFiles({ accept: 'image/*', multiple: true }).then((fileList) => {
        Array.from(fileList).forEach((item) => {
          getImgStr(item).then((file) => {
            this.insertImgFile(file);
          });
        });
      });
    },
    // 插入图片文件
    insertImgFile(file) {
      const imgEl = document.createElement('img');
      imgEl.src = file || this.imgFile;
      // 插入页面
      document.body.appendChild(imgEl);
      imgEl.onload = () => {
        // 创建图片对象
        const imgInstance = new this.fabric.Image(imgEl, {
          id: uuid(),
          name: '图片1',
          left: 100,
          top: 100,
        });
        // 设置缩放
        this.canvas.c.add(imgInstance);
        this.canvas.c.setActiveObject(imgInstance);
        this.canvas.c.renderAll();
        // 删除页面中的图片元素
        imgEl.remove();
      };
    },
  },
};
</script>

<style scoped lang="less">
:deep(.ivu-select-dropdown) {
  z-index: 999;
}
</style>
