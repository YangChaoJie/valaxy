<script lang="ts" setup>
import type { Ref } from 'vue'
import { inject, onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAplayer, useCodePen, useCopyCode, useMediumZoom, wrapTable } from 'valaxy'
import type { Post } from 'valaxy'

import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'

const props = defineProps<{
  frontmatter: Post
  excerpt?: string
}>()

dayjs.extend(relativeTime)

const onContentUpdated = inject('onContentUpdated') as Ref<() => void>

const { t } = useI18n()

const content = ref()
function updateDom() {
  wrapTable(content.value)
  onContentUpdated.value?.()
}

onMounted(() => {
  updateDom()
})

// widgets
if (props.frontmatter.aplayer)
  useAplayer()

if (props.frontmatter.codepen)
  useCodePen()

useCopyCode()

if (typeof props.frontmatter.medium_zoom === 'undefined' || props.frontmatter.medium_zoom)
  useMediumZoom()
</script>

<template>
  <article v-if="$slots.default" :class="frontmatter.markdown !== false && 'markdown-body'">
    <ValaxyMdTimeWarning :frontmatter="frontmatter" />
    <slot ref="content" @vnode-updated="updateDom" />

    <div text="center">
      <a
        v-if="frontmatter.url"
        :href="frontmatter.url"
        class="post-link-btn shadow hover:shadow-md"
        rounded
        target="_blank"
        m="b-4"
      >
        {{ t('post.view_link') }}
      </a>
    </div>

    <slot v-if="frontmatter.end !== undefined" name="end">
      <div m="y-4" class="end flex justify-center items-center">
        <hr class="line inline-flex" w="full" m="!y-2">
        <span p="x-4" font="serif bold" class="whitespace-nowrap">
          {{ frontmatter.end ? 'Q.E.D.' : 'To Be Continued.' }}
        </span>
        <hr class="line inline-flex" w="full" m="!y-2">
      </div>
    </slot>
  </article>
</template>

<style>
.medium-zoom-overlay,
.medium-zoom-image--opened {
  z-index: 999;
}
</style>
