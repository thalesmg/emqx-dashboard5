<template>
  <el-row>
    <el-col :span="24">
      <p class="desc-config tip">{{ blockTip }}</p>
    </el-col>
    <template v-for="rateProp in properties" :key="rateProp">
      <el-col v-if="rateProp.path" :span="21" class="col-custom-width">
        <el-form-item>
          <template #label>
            <FormItemLabel
              :label="tl(`emqx_limiter_schema.${rateProp.key}.label`)"
              :desc="tl(`emqx_limiter_schema.${rateProp.key}.desc`)"
              desc-marked
            />
          </template>
          <SchemaFormItem
            :type="rateProp.type"
            :disabled="rateProp.readOnly"
            :placeholder="rateProp.default.toString()"
            :model-value="getModelValue(formData, rateProp.path)"
            @update:model-value="setModelValue(formData, rateProp.path, $event)"
            :symbols="rateProp.symbols"
            :default-value="rateProp.default"
          />
        </el-form-item>
      </el-col>
    </template>
  </el-row>
</template>

<script setup lang="ts">
import FormItemLabel from '@/components/FormItemLabel.vue'
import SchemaFormItem from '@/components/SchemaFormItem'
import useI18nTl from '@/hooks/useI18nTl'
import useTwoWayBindingManually from '@/hooks/useTwoWayBindingManually'
import { computed, defineEmits, defineProps } from 'vue'

const props = defineProps({
  modelValue: {
    type: Object,
  },
  properties: {
    type: Object,
  },
  blockTip: {
    type: String,
  },
})

const emit = defineEmits(['update:modelValue'])

const { tl } = useI18nTl('Schema')

const { getModelValue, setModelValue } = useTwoWayBindingManually()

const formData = computed({
  get() {
    return props.modelValue
  },
  set(val) {
    emit('update:modelValue', val)
  },
})
</script>

<style lang="scss"></style>
