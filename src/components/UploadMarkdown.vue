<template>
  <div>
    <el-container>
      <el-header>
        <h1>Upload PDF and Get Markdown</h1>
      </el-header>
      <el-main>
        <div class="flex-container">
          <div class="upload-section">
            <el-upload
              class="upload-demo"
              drag
              :http-request="uploadFile"
              action="http://dummy-url"
              multiple
              :on-change="handleFileChange"
              :before-upload="beforeUpload">
              <i class="el-icon-upload"></i>
              <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
              <div class="el-upload__tip" slot="tip">只能上传PDF文件</div>
            </el-upload>
            <div v-if="loading" class="loading">Loading...</div>
            <div v-if="error" class="error">
              <el-alert :title="error" type="error" show-icon></el-alert>
            </div>
          </div>
          <div
            v-if="markdownContent"
            class="markdown"
            v-html="parsedMarkdownContent"></div>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import axios from 'axios'
import MarkdownIt from 'markdown-it'
import markdownItKatex from 'markdown-it-katex'

export default {
  name: 'UploadMarkdown',
  data() {
    return {
      selectedFile: null,
      markdownContent: '# Markdown content will be displayed here.', // 初始值为空字符串
      loading: false,
      error: null,
      markdownIt: null
    }
  },
  computed: {
    parsedMarkdownContent() {
      return this.renderMarkdown(this.markdownContent)
    }
  },
  created() {
    this.markdownIt = new MarkdownIt().use(markdownItKatex)
  },
  mounted() {
    const link = document.createElement('link')
    link.rel = 'stylesheet'
    link.href = 'https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.5.1/katex.min.css'
    document.head.appendChild(link)
  },
  methods: {
    handleFileChange(file) {
      this.selectedFile = file.raw
    },
    beforeUpload(file) {
      this.selectedFile = file
      // 直接调用 uploadFile 方法
      this.uploadFile({
        file,
        onProgress: () => {},
        onSuccess: () => {},
        onError: () => {}
      })
      return false // 阻止默认上传行为
    },
    async uploadFile({ file, onProgress, onSuccess, onError }) {
      const formData = new FormData()
      formData.append('file', file)

      this.loading = true
      this.error = null

      const loadingInstance = this.$loading({
        lock: true,
        text: 'Loading...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      try {
        const response = await axios.post(
          'http://192.168.64.22:5000/pdf-extract',
          formData,
          {
            headers: {
              'Content-Type': 'multipart/form-data'
            },
            onUploadProgress: (progressEvent) => {
              const percentCompleted = Math.round(
                (progressEvent.loaded * 100) / progressEvent.total
              )
              onProgress({ percent: percentCompleted })
            }
          }
        )
        console.log('Server response:', response.data) // 调试日志
        this.markdownContent = response.data.markdown || '' // 确保为字符串
        console.log('Updated markdownContent:', this.markdownContent) // 调试日志
        onSuccess(response.data)
      } catch (error) {
        this.error = 'Error uploading file: ' + error.message
        onError(error)
      } finally {
        this.loading = false
        loadingInstance.close()
      }
    },
    renderMarkdown(content) {
      return this.markdownIt.render(content)
    }
  }
}
</script>

<style scoped>
h1 {
  font-size: 24px;
  margin-bottom: 20px;
}

.flex-container {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.upload-section {
  flex: 1;
  margin-right: 20px;
}

.upload-demo {
  margin-bottom: 20px;
  border: 2px dashed #d9d9d9;
  border-radius: 6px;
  background-color: #fafafa;
  text-align: center;
  padding: 20px;
}

.loading,
.error {
  margin-top: 20px;
}

.markdown {
  flex: 2;
  width: 800px;
  margin: 0 auto;
}
</style>
