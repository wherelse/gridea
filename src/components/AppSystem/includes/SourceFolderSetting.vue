<template>
  <div class="mb-4 py-4 border-b border-gray-200">
    <div class="text-base font-medium mb-4">{{ $t('sourceFolder') }}</div>
    <a-form>
      <a-form-item :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
        <a-input v-model="currentFolderPath" read-only>
          <i slot="addonAfter" class="zwicon-folder-open px-2" @click="handleFolderSelect"></i>
        </a-input>
      </a-form-item>
      <a-form-item :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
        <a-button type="primary" @click="save">{{ $t('save') }}</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script lang="ts">
import {
  ipcRenderer, IpcRendererEvent,
} from 'electron'
import { app as remoteApp, dialog as remoteDialog } from '@electron/remote'
import { Vue, Component } from 'vue-property-decorator'
import { State } from 'vuex-class'

@Component
export default class System extends Vue {
  @State('site') site!: any

  formLayout = {
    label: { span: 5 },
    wrapper: { span: 12 },
  }

  currentFolderPath = '-'

  mounted() {
    this.currentFolderPath = this.site.appDir
  }

  save() {
    ipcRenderer.send('app-source-folder-setting', this.currentFolderPath)
    ipcRenderer.once('app-source-folder-set', (event: IpcRendererEvent, data: any) => {
      if (data) {
        this.$message.success(this.$t('saved'))
        this.$bus.$emit('site-reload')
        remoteApp.relaunch()
        remoteApp.quit()
      } else {
        this.$message.error(this.$t('saveError'))
      }
    })
  }

  async handleFolderSelect() {
    const res = await remoteDialog.showOpenDialog({
      properties: ['openDirectory', 'createDirectory'],
    })
    if (res.filePaths.length > 0) {
      this.currentFolderPath = res.filePaths[0].replace(/\\/g, '/')
    }
  }
}
</script>

<style lang="less" scoped>
/deep/ .ant-input-group-addon {
  padding: 0;
}
.folder-btn {
  padding: 0 8px;
}
</style>
