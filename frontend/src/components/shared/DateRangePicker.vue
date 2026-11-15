<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import type { TimeRangePreset } from '@/composables/useDateRange'

interface DayValue { year: number; month: number; day: number }
interface RangeValue { start?: DayValue; end?: DayValue }

// isDatePickerOpen and formatDateRangeDisplay are kept so callers' bindings stay valid;
// the native date inputs are always visible for a custom range, so neither is needed.
const props = defineProps<{
  selectedRange: TimeRangePreset
  customDateRange: RangeValue | null | undefined
  isDatePickerOpen?: boolean
  formatDateRangeDisplay?: string
}>()

const emit = defineEmits<{
  'update:selectedRange': [value: TimeRangePreset]
  'update:customDateRange': [value: RangeValue]
  'update:isDatePickerOpen': [value: boolean]
  'apply-custom': []
}>()

const { t } = useI18n()

const pad = (n: number) => String(n).padStart(2, '0')
const toIso = (d?: DayValue) => (d ? `${d.year}-${pad(d.month)}-${pad(d.day)}` : '')
const fromIso = (s: string): DayValue | undefined => {
  const [y, m, d] = s.split('-').map(Number)
  return s && y && m && d ? { year: y, month: m, day: d } : undefined
}

// Analytics only exist up to today; a future range is always empty.
const maxDate = (() => {
  const n = new Date()
  return `${n.getFullYear()}-${pad(n.getMonth() + 1)}-${pad(n.getDate())}`
})()

const from = computed(() => toIso(props.customDateRange?.start))
const to = computed(() => toIso(props.customDateRange?.end))

function setStart(e: Event) {
  emit('update:customDateRange', { start: fromIso((e.target as HTMLInputElement).value), end: props.customDateRange?.end })
}
function setEnd(e: Event) {
  emit('update:customDateRange', { start: props.customDateRange?.start, end: fromIso((e.target as HTMLInputElement).value) })
}
</script>

<template>
  <Select
    :model-value="selectedRange"
    @update:model-value="emit('update:selectedRange', $event as TimeRangePreset)"
  >
    <SelectTrigger class="w-[140px]">
      <SelectValue :placeholder="t('dateRange.selectRange', 'Date Range')" />
    </SelectTrigger>
    <SelectContent>
      <SelectItem value="all">{{ t('dateRange.all', 'All') }}</SelectItem>
      <SelectItem value="today">{{ t('dateRange.today', 'Today') }}</SelectItem>
      <SelectItem value="7days">{{ t('dateRange.last7Days', 'Last 7 Days') }}</SelectItem>
      <SelectItem value="30days">{{ t('dateRange.last30Days', 'Last 30 Days') }}</SelectItem>
      <SelectItem value="this_month">{{ t('dateRange.thisMonth', 'This Month') }}</SelectItem>
      <SelectItem value="custom">{{ t('dateRange.customRange', 'Custom Range') }}</SelectItem>
    </SelectContent>
  </Select>
  <Popover
    v-if="selectedRange === 'custom'"
    :open="isDatePickerOpen"
    @update:open="emit('update:isDatePickerOpen', $event)"
  >
    <PopoverTrigger as-child>
      <Button variant="outline" size="sm">
        <CalendarIcon class="h-4 w-4 mr-1" />
        {{ formatDateRangeDisplay || t('common.select') }}
      </Button>
    </PopoverTrigger>
    <PopoverContent class="w-auto p-4" align="end">
      <div class="space-y-4">
        <RangeCalendar
          :model-value="customDateRange"
          @update:model-value="emit('update:customDateRange', $event)"
          :number-of-months="2"
          :max-value="maxDate"
        />
        <Button
          class="w-full"
          size="sm"
          @click="emit('apply-custom')"
          :disabled="!customDateRange?.start || !customDateRange?.end"
        >
          {{ t('dateRange.apply', 'Apply') }}
        </Button>
      </div>
    </PopoverContent>
  </Popover>
</template>
