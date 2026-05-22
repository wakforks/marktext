<template>
  <div class="readonly-viewer">
    <div class="readonly-banner">
      <span>{{ filename }} — Read Only</span>
    </div>
    <div ref="cmContainer" class="readonly-cm-container" />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { usePreferencesStore } from '@/store/preferences'
import { storeToRefs } from 'pinia'
import codeMirror from '../../codeMirror'
import bus from '../../bus'
import { oneDarkThemes, railscastsThemes } from '@/config'

const props = defineProps({
  content: {
    type: String,
    required: true
  },
  filename: {
    type: String,
    required: true
  },
  pathname: {
    type: String,
    required: true
  }
})

const preferencesStore = usePreferencesStore()
const { theme } = storeToRefs(preferencesStore)

const cmContainer = ref(null)
let cmInstance = null

const getModeForFile = (filename) => {
  const info = codeMirror.findModeByFileName(filename)
  return info || null
}

const handleReadOnlyFileChanged = ({ content, pathname, filename }) => {
  if (!cmInstance) return
  cmInstance.setValue(content)
  const modeInfo = getModeForFile(filename)
  if (modeInfo) {
    codeMirror.requireMode(modeInfo.mode, () => {
      cmInstance.setOption('mode', modeInfo.mime || modeInfo.mode)
    })
  }
}

onMounted(() => {
  const container = cmContainer.value
  const cmConfig = {
    value: props.content,
    readOnly: true,
    lineNumbers: true,
    lineWrapping: true,
    styleActiveLine: false,
    cursorBlinkRate: -1,
    viewportMargin: Infinity,
    lineNumberFormatter (line) {
      if (line % 10 === 0 || line === 1) {
        return line
      } else {
        return ''
      }
    }
  }

  if (railscastsThemes.includes(theme.value)) {
    cmConfig.theme = 'railscasts'
  } else if (oneDarkThemes.includes(theme.value)) {
    cmConfig.theme = 'one-dark'
  }

  cmInstance = codeMirror(container, cmConfig)

  const modeInfo = getModeForFile(props.filename)
  if (modeInfo) {
    codeMirror.requireMode(modeInfo.mode, () => {
      cmInstance.setOption('mode', modeInfo.mime || modeInfo.mode)
    })
  }

  bus.on('readonly-file-changed', handleReadOnlyFileChanged)
})

onBeforeUnmount(() => {
  bus.off('readonly-file-changed', handleReadOnlyFileChanged)
  cmInstance = null
})
</script>

<style>
.readonly-viewer {
  height: calc(100vh - var(--titleBarHeight));
  box-sizing: border-box;
  overflow: auto;
  display: flex;
  flex-direction: column;
}
.readonly-banner {
  flex-shrink: 0;
  padding: 4px 16px;
  font-size: 12px;
  color: var(--editorColor50);
  background: var(--floatBgColor);
  border-bottom: 1px solid var(--itemBgColor);
  text-align: center;
  font-style: italic;
}
.readonly-cm-container {
  flex: 1;
  overflow: auto;
}
.readonly-cm-container .CodeMirror {
  height: 100%;
  background: transparent;
}
.readonly-cm-container .CodeMirror-gutters {
  border-right: none;
  background-color: transparent;
}
.readonly-cm-container .CodeMirror-cursor {
  display: none;
}
</style>
