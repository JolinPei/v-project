<template>
  <Tooltip v-if="mSelectMode" :content="$t('quick.del')">
    <Button @click="del" icon="ios-trash" type="text"></Button>
  </Tooltip>
</template>

<script>
import select from '@/mixins/select';

export default {
  name: 'ToolBar',
  mixins: [select],
  data() {
    return {};
  },
  methods: {
    del() {
      const activeObject = this.canvas.c.getActiveObjects();
      if (activeObject) {
        activeObject.map((item) => this.canvas.c.remove(item));
      }
      this.canvas.c.requestRenderAll();
      this.canvas.c.discardActiveObject();
    },
  },
};
</script>

<style scoped lang="less"></style>
