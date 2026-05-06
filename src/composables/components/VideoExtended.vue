<script lang="ts" setup>
import type { NodeViewProps } from '@tiptap/vue-3';
import { computed } from 'vue';
import { Video, Loader2 } from 'lucide-vue-next';
import MediaExtended from '../commands/MediaExtended.vue';
import { useMediaLoadStatus } from './useMediaLoadStatus';
import { t } from '../i18n';

const props = defineProps<NodeViewProps>();

const src = computed(() => String(props.node.attrs.src ?? ''));
const { status, markLoaded, markError } = useMediaLoadStatus(src);
</script>

<template>
  <MediaExtended v-bind="props" v-slot="{ setMediaRef }">
    <div
      :ref="setMediaRef"
      class="edra-media-asset-root relative w-full overflow-hidden rounded-md bg-[color-mix(in_srgb,var(--vtip-placeholder-text,#9ca3af)_14%,transparent)]"
      :class="status !== 'loaded' ? 'min-h-48' : ''"
    >
      <video
        v-show="status !== 'error'"
        :src="props.node.attrs.src"
        :title="props.node.attrs.title"
        controls
        class="relative z-0 w-full max-w-full"
        @canplay="markLoaded"
        @error="markError"
      />
      <div
        v-if="status === 'loading'"
        class="edra-media-placeholder edra-media-placeholder--video absolute inset-0 z-[1] flex flex-col items-center justify-center gap-2 bg-[color-mix(in_srgb,var(--vtip-placeholder-text,#9ca3af)_14%,transparent)] py-10 text-[var(--vtip-placeholder-text,#6b7280)]"
        aria-busy="true"
      >
        <Loader2 class="size-10 animate-spin opacity-70" aria-hidden="true" />
        <span class="text-sm">{{ t('mediaVideoLoading') }}</span>
      </div>
      <div
        v-else-if="status === 'error'"
        class="edra-media-placeholder edra-media-placeholder--video flex min-h-48 flex-col items-center justify-center gap-2 py-10 text-[var(--vtip-placeholder-text,#6b7280)]"
        role="status"
        :aria-label="t('mediaVideoAriaLoadFailed')"
      >
        <Video class="size-10 opacity-80" aria-hidden="true" />
        <span class="text-sm">{{ t('mediaVideoLoadFailed') }}</span>
      </div>
    </div>
  </MediaExtended>
</template>
