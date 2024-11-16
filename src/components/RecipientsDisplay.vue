<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import RecipientsBadge from './RecipientsBadge.vue'

const props = defineProps<{
  recipients: string[]
}>()

const containerRef = ref<HTMLDivElement | null>(null)
const recipientsRef = ref<HTMLSpanElement | null>(null)
const visibleRecipients = ref<string[]>([])
const numTruncated = ref(0)
const showTooltip = ref(false)

const handleBadgeMouseEnter = () => {
  showTooltip.value = true
}

const handleBadgeMouseLeave = () => {
  showTooltip.value = false
}

const updateVisibleRecipients = () => {
  if (!containerRef.value || !recipientsRef.value || props.recipients.length === 0) {
    return
  }

  const containerWidth = containerRef.value.offsetWidth
  const tempSpan = document.createElement('span')
  tempSpan.style.visibility = 'hidden'
  tempSpan.style.position = 'absolute'
  tempSpan.style.whiteSpace = 'nowrap'
  document.body.appendChild(tempSpan)

  // If only one recipient, show it (will be truncated by CSS if needed)
  if (props.recipients.length === 1) {
    visibleRecipients.value = [props.recipients[0]]
    numTruncated.value = 0
    document.body.removeChild(tempSpan)
    return
  }

  // For multiple recipients, always start with first recipient
  const firstRecipient = props.recipients[0]
  visibleRecipients.value = [firstRecipient]
  numTruncated.value = props.recipients.length - 1

  // Calculate badge width if needed
  const badgeWidth = props.recipients.length > 1 ? 45 : 0 // Approximate width of badge including margin

  // Try to fit more recipients
  let currentText = firstRecipient
  for (let i = 1; i < props.recipients.length; i++) {
    const nextRecipient = props.recipients[i]
    const isLastRecipient = i === props.recipients.length - 1
    // Only add ellipsis if it's not the last recipient
    const textToTest = `${currentText}, ${nextRecipient}${isLastRecipient ? '' : ', ...'}`
    tempSpan.textContent = textToTest

    if (tempSpan.offsetWidth + badgeWidth <= containerWidth) {
      visibleRecipients.value.push(nextRecipient)
      numTruncated.value = props.recipients.length - visibleRecipients.value.length
      currentText = visibleRecipients.value.join(', ')
    } else {
      break
    }
  }

  document.body.removeChild(tempSpan)
}

const resizeObserver = new ResizeObserver(() => {
  updateVisibleRecipients()
})

onMounted(() => {
  if (containerRef.value) {
    resizeObserver.observe(containerRef.value)
  }
  updateVisibleRecipients()
})

onUnmounted(() => {
  resizeObserver.disconnect()
})
</script>

<template>
  <div ref="containerRef" class="recipients-container">
    <span ref="recipientsRef" class="recipients">
      {{ visibleRecipients.join(', ')
      }}{{ visibleRecipients.length < recipients.length ? ', ...' : '' }}
    </span>
    <RecipientsBadge
      v-if="numTruncated > 0"
      :numTruncated="numTruncated"
      @mouseenter="handleBadgeMouseEnter"
      @mouseleave="handleBadgeMouseLeave"
    />
    <div v-if="showTooltip" class="recipients-tooltip">
      {{ recipients.join(', ') }}
    </div>
  </div>
</template>

<style scoped>
.recipients-container {
  display: flex;
  align-items: center;
  width: 100%;
  overflow: hidden;
}

.recipients {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-right: 4px;
  flex: 1;
}

.recipients-tooltip {
  position: fixed;
  top: 8px;
  right: 8px;
  padding: 8px 16px;
  background-color: #666;
  color: #f0f0f0;
  border-radius: 24px;
  display: flex;
  align-items: center;
  z-index: 1000;
}
</style>
