<template>
  <component :is="type" v-bind="linkProps(to)" @click="handleLinkClick">
    <slot />
  </component>
</template>

<script>
import { isExternal } from '@/utils/validate'
import { openExternalLinkWithToken } from '@/utils/external-link'

export default {
  props: {
    to: {
      type: [String, Object],
      required: true
    },
    menuMeta: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    isExternalLink() {
      return isExternal(this.to)
    },
    //内嵌显示的外链
    isEmbeddedDisplay() {
      // isFrame=0表示外链且frameEmbedFlag=1表示内嵌显示
      return (this.menuMeta.isFrame == '0' && this.menuMeta.frameEmbedFlag == '1')
    },
    // 新开窗口的外链
    isExternalJump() {
      // isFrame=0表示外链且frameEmbedFlag=2表示外置跳转
      return this.menuMeta.isFrame == '0' && this.menuMeta.frameEmbedFlag == '2'
    },
    type() {
      // 新开窗口的外链
      if (this.isExternalJump) {
        return 'a'
      }
      // 其他情况使用router-link
      return 'router-link'
    }
  },
  methods: {
    linkProps(to) {
      // 外置跳转使用a标签属性
      if (this.isExternalJump) {
        return {
          href: to,
          target: '_blank',
          rel: 'noopener'
        }
      }
      // 内嵌显示的外链，创建特殊路由路径
      if (this.isEmbeddedDisplay) {
        console.log('测试:', this.to);
        return {
          to: {
            path: '/iframe/' + encodeURIComponent(this.to),
            query: { // 使用 query 传递标题
              title: this.menuMeta.title || 'External Link'
            },
            meta: {
              title: this.menuMeta.title || 'External Link',
              link: this.to
            }
          }
        }
      }
      // 普通路由
      return {
        to: to
      }
    },
    handleLinkClick(event) {
      console.log('Link clicked:', {
        to: this.to,
        isExternalJump: this.isExternalJump,
        isEmbeddedDisplay: this.isEmbeddedDisplay,
        menuMeta: this.menuMeta
      })

      // 外置跳转处理
      if (this.isExternalJump) {
        event.preventDefault()
        console.log('External jump:', this.to)
        openExternalLinkWithToken(this.to)
      }
      // 内嵌显示的外链处理
      else if (this.isEmbeddedDisplay && this.menuMeta.isFrame == '0') {
        // 阻止默认的router-link行为
        event.preventDefault()

        // 对于需要内嵌显示的外链，添加到iframe视图
        const iframePath = '/iframe/' + encodeURIComponent(this.to)
        console.log('Iframe path:', iframePath)

        this.$store.dispatch('tagsView/addIframeView', {
          path: iframePath,
          name: this.menuMeta.title || 'External Link',
          meta: {
            title: this.menuMeta.title || 'External Link',
            link: this.to,
            isFrame: this.menuMeta.isFrame,
            frameEmbedFlag: this.menuMeta.frameEmbedFlag
          }
        })
        // 同时添加到访问视图
        this.$store.dispatch('tagsView/addVisitedView', {
          path: iframePath,
          name: this.menuMeta.title || 'External Link',
          meta: {
            title: this.menuMeta.title || 'External Link',
            link: this.to
          }
        })
        // 实际进行路由跳转
        console.log('Pushing to router:', iframePath)
        this.$router.push(iframePath).then(() => {
          console.log('Router push successful')
        }).catch(err => {
          console.error('Router push failed:', err)
        })
      }
      // 内嵌页面显示(isFrame=1)的情况由router-link自动处理
    }
  }
}
</script>
