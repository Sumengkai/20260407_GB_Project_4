<template>
  <div>
    <template v-if="showSelf">
      <OutputSection
        label="自產" color-class=""
        :rows="selfRows"
        @add="$emit('add', { section:'self', row:$event })"
        @update="$emit('update', { section:'self', idx:$event.idx, row:$event.row })"
        @delete-items="$emit('delete-items', { section:'self', ids:$event })"
        @open-inbound="$emit('open-inbound')"
      />
    </template>
    <template v-if="showOutsource">
      <OutputSection
        label="委外" color-class="outsource"
        :rows="outsourceRows"
        @add="$emit('add', { section:'outsource', row:$event })"
        @update="$emit('update', { section:'outsource', idx:$event.idx, row:$event.row })"
        @delete-items="$emit('delete-items', { section:'outsource', ids:$event })"
        @open-transfer="$emit('open-transfer')"
      />
    </template>
  </div>
</template>

<script setup>
import OutputSection from './OutputSection.vue'

defineProps({
  showSelf:       { type: Boolean, default: true },
  showOutsource:  { type: Boolean, default: true },
  selfRows:       { type: Array, default: () => [] },
  outsourceRows:  { type: Array, default: () => [] },
})
defineEmits(['add', 'update', 'delete-items', 'open-transfer', 'open-inbound'])
</script>
