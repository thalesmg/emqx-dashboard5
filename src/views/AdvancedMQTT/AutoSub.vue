<!-- Proxy Subscription -->
<template>
  <div class="app-wrapper subscribe">
    <div class="section-header">
      <div></div>
      <el-button type="primary" @click="openOpDialog()" :icon="Plus">
        {{ $t('Base.add') }}
      </el-button>
    </div>

    <el-table :data="subTbData" v-loading="tbLoading">
      <el-table-column :label="$t('Base.topic')" prop="topic" />
      <el-table-column :label="'QoS'" prop="qos" />
      <el-table-column :label="$t('Clients.noLocal')" prop="nl">
        <template #default="{ row }">
          {{ getLabelFromValueInOptionList(row.nl, noLocalOpts) }}
        </template>
      </el-table-column>
      <el-table-column :label="$t('Clients.retainAsPublished')" prop="rap">
        <template #default="{ row }">
          {{ getLabelFromValueInOptionList(row.rap, retainAsPublishedOpts) }}
        </template>
      </el-table-column>
      <el-table-column :label="$t('Clients.retainHandling')" prop="rh" />
      <el-table-column :label="$t('Base.operation')">
        <template #default="{ $index }">
          <el-button size="small" @click="openOpDialog($index)">{{ $t('Base.edit') }}</el-button>
          <el-button size="small" plain @click="deleteSubs($index)">
            {{ $t('Base.delete') }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
  <el-dialog
    v-model="opSubs"
    :title="isEdit ? $t('Base.edit') : $t('Base.add')"
    @close="closeDialog"
  >
    <TipContainer :content="$t('Extension.proxySubTip')" />
    <el-form
      :rules="subsRules"
      :model="subsInput"
      ref="subsForm"
      label-position="top"
      require-asterisk-position="right"
      @keyup.enter="submitSubs(isEdit)"
    >
      <el-form-item :label="$t('Base.topic')" prop="topic">
        <el-input v-model="subsInput.topic" />
      </el-form-item>
      <el-form-item :label="'QoS'" prop="qos">
        <el-select v-model="subsInput.qos">
          <el-option v-for="item in QoSOptions" :key="item" :value="item" />
        </el-select>
      </el-form-item>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-form-item :label="$t('Clients.noLocal')" prop="nl">
            <el-select v-model="subsInput.nl">
              <el-option
                v-for="{ value, label } in noLocalOpts"
                :label="label"
                :value="value"
                :key="value"
              />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item :label="$t('Clients.retainAsPublished')" prop="rap">
            <el-select v-model="subsInput.rap">
              <el-option
                v-for="{ value, label } in retainAsPublishedOpts"
                :label="label"
                :value="value"
                :key="value"
              />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item :label="$t('Clients.retainHandling')" prop="rh">
            <el-select v-model="subsInput.rh">
              <el-option
                v-for="item in retainHandlingOpts"
                :label="item"
                :value="item"
                :key="item"
              />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <el-button @click="closeDialog()">{{ $t('Base.cancel') }}</el-button>
      <el-button type="primary" @click="submitSubs(isEdit)" :loading="submitLoading">
        {{ isEdit ? $t('Base.update') : $t('Base.add') }}
      </el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { editSubscribe, getSubscribe } from '@/api/extension'
import { QoSOptions } from '@/common/constants'
import { getLabelFromValueInOptionList } from '@/common/tools'
import TipContainer from '@/components/TipContainer.vue'
import useI18nTl from '@/hooks/useI18nTl'
import useMQTTVersion5NewConfig from '@/hooks/useMQTTVersion5NewConfig'
import { QoSLevel } from '@/types/enum'
import { AutoSubscribe } from '@/types/extension'
import { Plus } from '@element-plus/icons-vue'
import { ElMessage, ElMessageBox as MB } from 'element-plus'
import { nextTick, onMounted, ref, Ref } from 'vue'
import { useI18n } from 'vue-i18n'

const createRawSubForm = (): AutoSubscribe => ({
  topic: '',
  qos: QoSLevel.QoS0,
  nl: 0,
  rap: 0,
  rh: 0,
})

const { t } = useI18n()
const { tl } = useI18nTl('Extension')
let isEdit = ref(false)
let opSubs = ref(false)
let subTbData: Ref<Array<AutoSubscribe>> = ref([])
const { noLocalOpts, retainAsPublishedOpts, retainHandlingOpts } = useMQTTVersion5NewConfig()
let subsInput = ref(createRawSubForm())
let editPos: Ref<undefined | number> = ref(undefined)
let submitLoading = ref(false)
let tbLoading = ref(false)
let subsForm = ref()
let subsRules = {
  topic: [
    {
      required: true,
      message: tl('required'),
      trigger: ['blur'],
    },
  ],
}

let openOpDialog = async (editIndex?: number) => {
  opSubs.value = true
  isEdit.value = editIndex !== undefined
  const target = isEdit.value ? subTbData.value[editIndex as number] : undefined
  subsInput.value = target || createRawSubForm()
  editPos.value = editIndex
  await nextTick()
  subsForm.value?.clearValidate()
}

const submitSubs = async function (edit = false) {
  try {
    await subsForm.value?.validate()
    submitLoading.value = true
    let pendingTbData = [...subTbData.value]
    if (!edit) {
      pendingTbData.push(subsInput.value)
    } else {
      editPos.value !== undefined && pendingTbData.splice(editPos.value, 1, { ...subsInput.value })
    }
    await editSubscribe(pendingTbData)
    ElMessage.success(edit ? t('Base.updateSuccess') : t('Base.createSuccess'))
    loadData()
    opSubs.value = false
    editPos.value = undefined
  } catch (error) {
    //
  } finally {
    submitLoading.value = false
  }
}

const deleteSubs = async function (index: number) {
  try {
    await MB.confirm(t('Base.confirmDelete'), {
      confirmButtonText: t('Base.confirm'),
      cancelButtonText: t('Base.cancel'),
      confirmButtonClass: 'confirm-danger',
      type: 'warning',
    })
    let pendingTbData = [...subTbData.value]
    pendingTbData.splice(index, 1)
    await editSubscribe(pendingTbData)
    ElMessage.success(t('Base.deleteSuccess'))
    loadData()
  } catch (error) {
    //
  }
}

const closeDialog = () => {
  opSubs.value = false
  subsInput.value = createRawSubForm()
}

let loadData = async () => {
  try {
    tbLoading.value = true
    subTbData.value = await getSubscribe()
  } catch (error) {
    //
  } finally {
    tbLoading.value = false
  }
}

onMounted(loadData)
</script>

<style lang="scss" scoped>
.el-form:not(:first-child) {
  margin-top: 12px;
}
</style>
