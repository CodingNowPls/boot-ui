<template>
  <div class="app-container">
    <div id="hiprintDesigner" class="designer-container"></div>
    <el-button @click="saveTemplate">保存模板</el-button>
  </div>
</template>

<script>
import { hiprint } from 'vue-plugin-hiprint'
import { saveTemplate } from '@/api/print/print'

export default {
  data() {
    return {
      hiprintInstance: null,
      templateData: null
    }
  },
  mounted() {
    this.initDesigner()
  },
  methods: {
    initDesigner() {
      this.hiprintInstance = new hiprint.PrintDesigner({
        template: {}, // 初始空模板
        settingContainer: '#hiprintDesigner',
        options: {
          // 自定义模板元素
          elements: [
            { name: 'title', options: { title: '默认标题', field: 'title' }},
            { name: 'table', options: { columns: [] }}
          ]
        }
      })
    },
    async saveTemplate() {
      const templateJson = this.hiprintInstance.getTemplate()
      const { data } = await saveTemplate({
        name: '订单模板',
        config: templateJson
      })
      this.$message.success('模板保存成功')
    }
  }
}
</script>

<style scoped>
.designer-container {
  height: 800px;
  border: 1px solid #ddd;
}
</style>
