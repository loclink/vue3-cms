<template>
  <div class="write">
    <search-page
      title="文章查询："
      class="common-piece"
      :searchConfig="writeSearchConfig"
      :pageName="pageName"
    ></search-page>

    <content-page
      ref="contentPageRef"
      class="common-piece"
      title="文章列表："
      :contentConfig="writeContentConfig"
      :pageName="pageName"
      @handleCreate="handleCreate"
      @handleEditWrite="handleEditWrite"
      @handleEdit="handleEdit"
      @handlePreview="handlePreview"
    >
      <template #cover="{ row }">
        <el-image
          v-if="row.cover && row.cover.length"
          :src="row.cover"
          fit="fill"
          lazy
          alt="文章封面"
          class="essay-cover"
        ></el-image>
      </template>

      <template #write_tag="{ row }">
        <div v-if="row.write_tag.length">
          <template v-for="item of row.write_tag" :key="item">
            <el-tag size="mini" :style="{ marginLeft: '5px', marginBottom: '3px' }">{{
              $store.getters['blog/getTagName'](item)
            }}</el-tag>
          </template>
        </div>

        <div v-else>—</div>
      </template>

      <template #status="{ row }">
        <el-button :type="row.status ? 'success' : 'info'" @click="handleSwitchStatus(row)">
          {{ row.status ? '已发布' : '未发布' }}
        </el-button>
      </template>
    </content-page>

    <form-dialog
      :pageName="pageName"
      :dialogConfig="writeDialogConfig"
      :dialogFormData="dialogFormData"
      v-model="isShowDialog"
      ref="formDialogRef"
    ></form-dialog>
  </div>
</template>

<script lang="ts">
import SearchPage from '@/components/search-page/src/search-page.vue'
import ContentPage from '@/components/content-page/src/content-page.vue'
import FormDialog from '@/components/form-dialog/src/form-dialog.vue'

import { defineComponent, ref } from 'vue'

import { BASE_URL } from '@/service/request/config'
import { writeSearchConfig } from './config/search.config'
import { writeContentConfig } from './config/content.config'
import { writeDialogConfig } from './config/dialog.config'
import { useStore } from '@/store'
import { useRouter } from 'vue-router'

import { alterFormConfig, handleSelectOptions } from '@/utils'

import { usePageDialog } from '@/hooks/use-page-dialog'

export default defineComponent({
  components: {
    SearchPage,
    ContentPage,
    FormDialog
  },
  setup() {
    const pageName = 'write'
    const uploadIconPath = BASE_URL + '/files/cover'
    const isShowPreview = ref(false)
    const store = useStore()
    const selectOptions = ref()
    const router = useRouter()
    const contentPageRef = ref<InstanceType<typeof ContentPage>>()

    // 定义路由跳转方法
    const jumpRouter = (path: any, id: any) => {
      router.push({
        path: `/${path}/${id}`
      })
    }

    // 处理编辑文章内容按钮
    const handleEditWrite = (row: any) => {
      jumpRouter('editor', row.id)
    }

    // 处理预览按钮
    const handlePreview = (row: any) => {
      jumpRouter('preview', row.id)
    }

    // 切换发表状态
    const handleSwitchStatus = (row: any) => {
      store
        .dispatch('blog/switchPublishWriteAction', { id: row.id, status: row.status ? 0 : 1 })
        .then(() => {
          contentPageRef.value?.getPageListData()
        })
    }

    // 请求标签数据，修改配置参数
    store.dispatch('blog/writeTagDataAction').then((res) => {
      selectOptions.value = handleSelectOptions(res.data.list, 'writeTag')
      alterFormConfig(writeDialogConfig, 'write_tag', 'options', selectOptions.value)
    })

    // 修改图片上传相关参数
    writeDialogConfig.formItemConfig.find((item: any) => {
      if (item.field === 'cover') {
        item.avatarOption!.action = uploadIconPath
      }
    })

    const [handleCreate, handleEdit, isShowDialog, formDialogRef, dialogFormData] = usePageDialog()

    return {
      writeSearchConfig,
      writeContentConfig,
      writeDialogConfig,
      handleCreate,
      handleEdit,
      isShowDialog,
      formDialogRef,
      dialogFormData,
      pageName,
      isShowPreview,
      selectOptions,
      contentPageRef,
      handleEditWrite,
      handlePreview,
      handleSwitchStatus
    }
  }
})
</script>

<style scoped lang="less">
.essay-cover {
  height: 100px;
  border-radius: 5px;
}
.preview-content {
  border-radius: 5px;
  height: 500px;
  overflow: hidden;
  box-sizing: border-box;
  z-index: 999;
}
</style>
