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
    }
  },
  computed: {
    isExternal() {
      return isExternal(this.to)
    },
    type() {
      if (this.isExternal) {
        return 'a'
      }
      return 'router-link'
    }
  },
  methods: {
    linkProps(to) {
      if (this.isExternal) {
        return {
          href: to,
          target: '_blank',
          rel: 'noopener'
        }
      }
      return {
        to: to
      }
    },
    handleLinkClick(event) {
      if (this.isExternal) {
        event.preventDefault()
        openExternalLinkWithToken(this.to)
      }
    }
  }
}
</script>
