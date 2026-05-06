<script setup lang="ts">
import { NodeViewWrapper, type NodeViewProps } from '@tiptap/vue-3';
import { ref } from 'vue';
import { Link, UploadCloud, Video, Loader2 } from 'lucide-vue-next';
import { FileType } from '../utils';
import { useScreen } from '../useScreen';

const props = defineProps<NodeViewProps>();
const { editor } = props;
const { isMaxSm } = useScreen();
const canLocalUpload = editor.storage.fileDrop.supportsLocalUpload;

const videoUrl = ref('');
const isUploading = ref(false);

function handleSubmit(e: Event) {
  e.preventDefault();
  if (videoUrl.value) {
    editor.chain().focus().setVideo({ src: videoUrl.value }).run();
  }
}

async function openFileDialog() {
  isUploading.value = true;
  try {
    const fileDrop = await editor.storage.fileDrop.localFileGetter(FileType.VIDEO);
    if (fileDrop) {
      editor.chain().focus().setVideo({ src: fileDrop }).run();
      return;
    }
  } catch (error) {
    console.error(error);
  } finally {
    isUploading.value = false;
  }
}
</script>

<template>
  <NodeViewWrapper
    class="media-placeholder-container relative w-full min-w-0 group my-2 sm:my-4"
    contenteditable="false"
  >
    <!-- Read-only State -->
    <div
      v-if="!editor.isEditable"
      :class="[
        'media-placeholder-glass flex flex-col items-center justify-center w-full text-[var(--vtip-placeholder-text)]',
        isMaxSm ? 'min-h-[120px] py-5' : 'h-[150px]'
      ]"
    >
      <Video
        :class="['opacity-55 drop-shadow-sm', isMaxSm ? 'w-9 h-9 mb-2' : 'w-10 h-10 mb-3']"
        stroke-width="1.5"
      />
      <span class="text-sm font-medium tracking-tight">No video selected</span>
    </div>

    <div v-else class="media-placeholder-glass w-full min-w-0 max-w-full">
      <div
        :class="[
          'media-placeholder-glass__inner flex min-w-0',
          isMaxSm ? 'flex-col w-full h-auto' : 'flex-row w-full max-w-2xl h-[150px]'
        ]"
      >
        <!-- Uploading Overlay -->
        <div
          v-if="isUploading"
          class="media-placeholder-glass__overlay absolute inset-0 z-10 flex flex-col items-center justify-center rounded-[inherit]"
        >
          <Loader2
            class="w-10 h-10 text-[var(--vtip-code-function)] animate-spin mb-3 drop-shadow-sm"
            stroke-width="2"
          />
          <span class="text-sm font-medium text-[var(--vtip-placeholder-text)]">
            Uploading...
          </span>
        </div>

        <!-- Left/Top: Upload -->
        <div
          v-if="canLocalUpload"
          :class="[
            'media-placeholder-glass__pane--upload flex flex-col items-center justify-center relative min-w-0',
            isMaxSm ? 'flex-none p-3' : 'flex-1 min-w-0 p-6'
          ]"
        >
          <button
            type="button"
            :class="[
              'vtip-btn media-placeholder-glass__upload-btn flex flex-col shrink-0',
              isMaxSm
                ? 'w-full gap-2 py-3 min-h-0'
                : 'gap-2.5 py-5 w-full max-w-[200px] h-full min-h-0'
            ]"
            :disabled="isUploading"
            @click="openFileDialog"
          >
            <UploadCloud
              :class="[
                'opacity-90 shrink-0 text-[var(--vtip-code-function)]',
                isMaxSm ? 'w-8 h-8' : 'w-9 h-9'
              ]"
              stroke-width="1.5"
            />
            <span class="text-xs font-semibold uppercase tracking-wider opacity-90">Upload Video</span>
          </button>
        </div>

        <!-- Right/Bottom: Link -->
        <div
          :class="[
            'link-section flex flex-col items-center justify-center min-w-0 overflow-hidden',
            isMaxSm ? 'flex-none p-3' : 'flex-1 p-4 sm:p-6'
          ]"
        >
          <form
            :class="[
              'link-form flex flex-col w-full min-w-0 max-w-full sm:max-w-[260px] overflow-hidden',
              isMaxSm ? 'gap-2' : 'gap-3'
            ]"
            @submit="handleSubmit"
          >
            <div
              :class="[
                'link-form__input media-placeholder-glass__field flex items-center gap-2 min-h-[44px] px-3',
                isUploading ? 'media-placeholder-glass__field--disabled' : ''
              ]"
            >
              <Link class="w-4 h-4 text-[var(--vtip-placeholder-text)] shrink-0 opacity-80" />
              <input
                v-model="videoUrl"
                :disabled="isUploading"
                placeholder="Paste video URL..."
                required
                type="url"
                class="link-form__field flex-1 bg-transparent border-none outline-none text-sm text-[var(--vtip-slash-item-text)] placeholder-[var(--vtip-placeholder-text)] min-h-[28px]"
                @keydown.stop
              />
            </div>
            <button
              type="submit"
              class="vtip-btn vtip-btn--primary media-placeholder-glass__primary w-full min-h-[44px] py-2.5 flex-shrink-0"
              :disabled="isUploading"
              title="Embed Video"
            >
              <Link class="w-4 h-4" />
            </button>
          </form>
        </div>
      </div>
    </div>
  </NodeViewWrapper>
</template>

<style scoped>
.media-placeholder-container * {
  transition-property: color, background-color, border-color, opacity, transform, box-shadow;
  transition-duration: 200ms;
}

.link-section {
  max-width: 100%;
}

.link-form__input {
  min-width: 0;
}

.link-form__field {
  min-width: 0;
}
</style>
