<script setup lang="ts">
import { useSlideContext } from '@slidev/client'
import { parseTimeString } from '@slidev/parser/utils'
import { computed } from 'vue'

const { $frontmatter, $renderContext } = useSlideContext()

const raw = computed(() => {
  const fm = $frontmatter.value as any
  return fm?.time ?? fm?.pace ?? null
})

function formatSeconds(seconds: number) {
  const s = Math.max(0, Math.floor(seconds))
  const h = Math.floor(s / 3600)
  const m = Math.floor((s % 3600) / 60)
  const ss = s % 60
  if (h)
    return `${h}:${String(m).padStart(2, '0')}:${String(ss).padStart(2, '0')}`
  return `${m}:${String(ss).padStart(2, '0')}`
}

const label = computed(() => {
  if (!raw.value)
    return null
  try {
    const parsed = parseTimeString(String(raw.value))
    return formatSeconds(parsed.seconds)
  }
  catch {
    return String(raw.value)
  }
})

const show = computed(() =>
  $renderContext.value === 'presenter'
  && !!label.value,
)
</script>

<template>
  <div
    v-if="show"
    class="fixed bottom-3 right-3 px-3 py-2 rounded slidev-glass-effect text-white pointer-events-none select-none"
  >
    <div class="text-[10px] uppercase tracking-wide opacity-70">
      Target
    </div>
    <div class="text-sm font-mono leading-5">
      {{ label }}
    </div>
  </div>
</template>

