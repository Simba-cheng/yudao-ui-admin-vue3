<!-- 某个记录的跟进记录列表，目前主要用于 CRM 客户、商机等详情界面 -->
<template>
  <!-- 操作栏 -->
  <el-row class="mb-10px" justify="end">
    <el-button @click="openForm">
      <Icon class="mr-5px" icon="ep:edit" />
      写跟进
    </el-button>
  </el-row>
  <!-- 列表 -->
  <ContentWrap>
    <el-table v-loading="loading" :data="list" :show-overflow-tooltip="true" :stripe="true">
      <el-table-column
        :formatter="dateFormatter"
        align="center"
        label="创建时间"
        prop="createTime"
        width="180px"
      />
      <el-table-column align="center" label="跟进人" prop="creatorName" />
      <el-table-column align="center" label="跟进类型" prop="type">
        <template #default="scope">
          <dict-tag :type="DICT_TYPE.CRM_FOLLOW_UP_TYPE" :value="scope.row.type" />
        </template>
      </el-table-column>
      <el-table-column align="center" label="跟进内容" prop="content" />
      <el-table-column label="图片" align="center">
        <template #default="scope">
          <div v-if="scope.row.picUrls && scope.row.picUrls.length > 0" class="flex">
            <el-image
              v-for="(url, index) in scope.row.picUrls"
              :key="index"
              :src="url"
              :preview-src-list="scope.row.picUrls"
              class="w-10 h-10 mr-1"
              :initial-index="index"
              fit="cover"
              preview-teleported
            />
          </div>
        </template>
      </el-table-column>
      <el-table-column label="附件" align="center">
        <template #default="scope">
          <div v-if="scope.row.fileUrls && scope.row.fileUrls.length > 0" class="flex flex-col">
            <el-link
              v-for="(url, index) in scope.row.fileUrls"
              :key="index"
              :href="url"
              type="primary"
              target="_blank"
              download
            >
              {{ getFileName(url) }}
            </el-link>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        :formatter="dateFormatter"
        align="center"
        label="下次联系时间"
        prop="nextTime"
        width="180px"
      />
      <el-table-column
        align="center"
        label="关联联系人"
        prop="contactIds"
        v-if="bizType === BizTypeEnum.CRM_CUSTOMER"
      >
        <template #default="scope">
          <el-link
            v-for="contact in scope.row.contacts"
            :key="`key-${contact.id}`"
            :underline="false"
            type="primary"
            @click="openContactDetail(contact.id)"
            class="ml-5px"
          >
            {{ contact.name }}
          </el-link>
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        label="关联商机"
        prop="businessIds"
        v-if="bizType === BizTypeEnum.CRM_CUSTOMER"
      >
        <template #default="scope">
          <el-link
            v-for="business in scope.row.businesses"
            :key="`key-${business.id}`"
            :underline="false"
            type="primary"
            @click="openBusinessDetail(business.id)"
            class="ml-5px"
          >
            {{ business.name }}
          </el-link>
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作">
        <template #default="scope">
          <el-button link type="danger" @click="handleDelete(scope.row.id)"> 删除 </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <Pagination
      v-model:limit="queryParams.pageSize"
      v-model:page="queryParams.pageNo"
      :total="total"
      @pagination="getList"
    />
  </ContentWrap>

  <!-- 表单弹窗：添加/修改 -->
  <FollowUpRecordForm ref="formRef" @success="getList" />
</template>

<script lang="ts" setup>
import { dateFormatter } from '@/utils/formatTime'
import { DICT_TYPE } from '@/utils/dict'
import { FollowUpRecordApi, FollowUpRecordVO } from '@/api/crm/followup'
import FollowUpRecordForm from './FollowUpRecordForm.vue'
import { BizTypeEnum } from '@/api/crm/permission'

/** 跟进记录列表 */
defineOptions({ name: 'FollowUpRecord' })

const getFileName = (url: string) => {
  if (!url) {
    return ''
  }
  return url.substring(url.lastIndexOf('/') + 1)
}

const props = defineProps<{
  bizType: number
  bizId: number
}>()
const message = useMessage() // 消息弹窗
const { t } = useI18n() // 国际化

const loading = ref(true) // 列表的加载中
const list = ref<FollowUpRecordVO[]>([]) // 列表的数据
const total = ref(0) // 列表的总页数
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  bizType: 0,
  bizId: 0
})

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await FollowUpRecordApi.getFollowUpRecordPage(queryParams)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}

/** 添加/修改操作 */
const formRef = ref<InstanceType<typeof FollowUpRecordForm>>()
const openForm = () => {
  formRef.value?.open(props.bizType, props.bizId)
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    // 发起删除
    await FollowUpRecordApi.deleteFollowUpRecord(id)
    message.success(t('common.delSuccess'))
    // 刷新列表
    await getList()
  } catch {}
}

/** 打开联系人详情 */
const { push } = useRouter()
const openContactDetail = (id: number) => {
  push({ name: 'CrmContactDetail', params: { id } })
}

/** 打开商机详情 */
const openBusinessDetail = (id: number) => {
  push({ name: 'CrmBusinessDetail', params: { id } })
}

watch(
  () => props.bizId,
  () => {
    queryParams.bizType = props.bizType
    queryParams.bizId = props.bizId
    getList()
  }
)
</script>
