<template>
  <div class="common-pagination">
    <el-pagination
      v-if="meta.count && meta.count > defaultPageSizeOpt[0]"
      background
      layout="total, sizes, prev, pager, next"
      :page-sizes="defaultPageSizeOpt"
      v-model:page-size="meta.limit"
      v-model:current-page="meta.page"
      :total="meta.count"
      @size-change="handleSizeChanged"
      @current-change="handleCurrentChanged"
    />
    <MiniPagination
      v-else-if="meta.count === -1"
      :current-page="meta.page"
      :hasnext="meta.hasnext as boolean"
      @current-change="handleCurrentChanged"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, watch, PropType, defineProps, defineEmits } from 'vue'
import MiniPagination from './MiniPagination.vue'
import { PageData } from '@/types/common'

const props = defineProps({
  metaData: {
    type: Object as PropType<PageData>,
    required: true,
    default: () => ({}),
  },
})

const meta = computed<PageData>(() => props.metaData)
meta.value.limit ||= 20
meta.value.page ||= 1

const defaultPageSizeOpt = [20, 50, 100, 500]

const emits = defineEmits(['loadPage', 'update:metaData'])

watch(meta, (v) => {
  emits('update:metaData', v)
})

const handleSizeChanged = (size: number) => {
  meta.value.page = 1
  emits('loadPage', {
    page: meta.value.page,
    limit: size,
  })
}

const handleCurrentChanged = (current: number) => {
  if (meta.value.page !== current) {
    meta.value.page = current
  }
  emits('loadPage', {
    page: current,
    limit: meta.value.limit,
  })
}
</script>
