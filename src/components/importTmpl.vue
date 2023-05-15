<template>
  <div style="display: inline-block">
    <import-file></import-file>
  </div>
</template>

<script>
import select from '@/mixins/select';
import { downFontByJSON } from '@/utils/utils';
import axios from 'axios';
import importFile from '@/components/importFile.vue';
const repoSrc = import.meta.env.APP_REPO;
export default {
  name: 'ImportTmpl',
  mixins: [select],
  data() {
    return {
      jsonFile: null,
    };
  },
  components: {
    importFile,
  },
  created() {
    this.getTempList();
  },
  methods: {
    // 插入文件
    insertSvgFile() {
      this.$Spin.show({
        render: (h) => h('div', this.$t('alert.loading_fonts')),
      });

      downFontByJSON(this.jsonFile)
        .then(() => {
          this.$Spin.hide();
          this.canvas.c.loadFromJSON(this.jsonFile, () => {
            this.canvas.c.renderAll.bind(this.canvas.c);
            setTimeout(() => {
              const workspace = this.canvas.c.getObjects().find((item) => item.id === 'workspace');
              workspace.set('selectable', false);
              workspace.set('hasControls', false);
              this.canvas.c.requestRenderAll();
              this.canvas.editor.editorWorkspace.setSize(workspace.width, workspace.height);
              this.canvas.c.renderAll();
              this.canvas.c.requestRenderAll();
            }, 100);
          });
        })
        .catch(() => {
          this.$Spin.hide();
          this.$Message.error(this.$t('alert.loading_fonts_failed'));
        });
    },
    // 获取模板列表数据
    getTempList() {
      this.$Spin.show({
        render: (h) => h('div', this.$t('alert.loading_data')),
      });
      const getTemp = axios.get(`${repoSrc}template/index.json`);
      getTemp
        .then((res) => {
          this.list = res.data.data.map((item) => {
            item.tempUrl = repoSrc + item.tempUrl;
            item.src = repoSrc + item.src;
            return item;
          });
          this.$Spin.hide();
        })
        .catch(this.$Spin.hide);
    },
    // 获取模板数据
    getTempData(tmplUrl) {
      this.$Spin.show({
        render: (h) => h('div', this.$t('alert.loading_data')),
      });
      const getTemp = axios.get(tmplUrl);
      getTemp.then((res) => {
        this.jsonFile = JSON.stringify(res.data);
        this.insertSvgFile();
      });
    },
  },
};
</script>

<style scoped lang="less">
.tmpl-img {
  width: 94px;
  cursor: pointer;
  margin-right: 5px;
}
</style>
