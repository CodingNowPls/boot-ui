<template>
  <div class="app-container">
    <div id="printContainer"></div>
    <el-button @click="doPrint">立即打印</el-button>
  </div>
</template>

<script>
import { hiprint } from 'vue-plugin-hiprint'
import { getTemplateList } from '@/api/print/print'

export default {
  data() {
    return {
      printer: null,
      templateId: this.$route.query.templateId
    }
  },
  async mounted() {
    await this.loadTemplate()
  },
  methods: {
    async loadTemplate() {
      const { data } = await getTemplateList(this.templateId)
      this.printer = new hiprint.PrintTemplate({
        template: data.config
      })
      // 渲染数据
      this.printer.print({
        title: '订单001',
        tableData: [{ id: 1, name: '商品A' }]
      }, '#printContainer')
    },
    doPrint() {
      this.printer.print(null, '#printContainer', { printer: 'default' })
    }
  }
}
</script>
