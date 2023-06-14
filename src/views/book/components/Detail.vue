<template>
  <div class="detail">
    <el-form ref="postForm" :model="postForm" :rules="rules" class="form-container">
      <sticky :z-index="10" :class-name="'sub-navbar'">
        <el-button v-if="!isEdit" @click.prevent.stop="showGuide">显示帮助</el-button>
        <el-button v-loading="loading" style="margin-left: 10px;" type="success" @click="submitForm">
          {{ isEdit ? '编辑电子书' : '新增电子书' }}
        </el-button>
      </sticky>
      <div class="detail-container">
        <el-row>
          <Warning />
          <el-col :span="24">
            <ebook-upload
              :file-list="fileList"
              :disabled="isEdit"
              @onSuccess="onUploadSuccess"
              @onRemove="onUploadRemove"
            />
          </el-col>
          <el-col :span="24">
            <el-form-item style="margin-bottom: 40px;" prop="title">
              <MdInput v-model="postForm.title" :maxlength="100" name="name" required>
                书名
              </MdInput>
            </el-form-item>
            <div>
              <el-row>
                <el-col :span="12">
                  <el-form-item prop="author" label="作者：" :label-width="labelwidth">
                    <el-input v-model="postForm.author" placeholder="作者" />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop="publisher" label="出版社：" :label-width="labelwidth">
                    <el-input v-model="postForm.publisher" placeholder="出版社" />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop="language" label="语言：" :label-width="labelwidth">
                    <el-input v-model="postForm.language" placeholder="语言" />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop='rootFile' label="根文件：" :label-width="labelwidth">
                    <el-input v-model="postForm.rootFile" placeholder="根文件" disabled />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop='filePath' label="文件路径：" :label-width="labelwidth">
                    <el-input v-model="postForm.filePath" placeholder="文件路径" disabled />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop='unzipPath' label="解压路径：" :label-width="labelwidth">
                    <el-input v-model="postForm.unzipPath" placeholder="解压路径" disabled />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop='coverPath' label="封面路径：" :label-width="labelwidth">
                    <el-input v-model="postForm.coverPath" placeholder="封面路径" disabled />
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item prop='originalName' label="文件名称：" :label-width="labelwidth">
                    <el-input v-model="postForm.originalName" placeholder="文件名称" disabled />
                  </el-form-item>
                </el-col>
                <el-col :span="24">
                  <el-form-item prop='cover' label="封面：" :label-width="labelwidth">
                    <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                      <img :src="postForm.cover" class="preview-img">
                    </a>
                    <span v-else>无</span>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row>
                <el-col>
                  <el-form-item label="目录：" :label-width="labelwidth">
                    <div v-if="contentsTree && contentsTree.length > 0" class="contents-wrapper">
                      <el-tree :data="contentsTree" @node-click="onContentClick" />
                    </div>
                    <span v-else>无</span>
                  </el-form-item>
                </el-col>
              </el-row>
            </div>
          </el-col>
        </el-row>
      </div>
    </el-form>
  </div>
</template>

<script>
import Sticky from '../../../components/Sticky'
import Warning from './Warning'
import EbookUpload from '../../../components/EbookUpload'
import MdInput from '../../../components/MDinput/index.vue'
import { createBook, getBook, updateBook } from '../../../api/book'

const defaultForm = {
  title: '',
  author: '',
  publisher: '',
  language: '',
  rootFile: '',
  cover: '',
  url: '',
  originalName: '',
  fileName: '',
  coverPath: '',
  filePath: '',
  unzipPath: ''
}

const fields = {
  title: '书名',
  author: '作者',
  publisher: '出版社',
  language: '语言'
}

export default {
  components: { MdInput, Sticky, Warning, EbookUpload },
  props: {
    isEdit: Boolean
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value.length === 0) {
        callback(new Error(fields[rule.field] + '必须填写'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      postForm: {},
      fileList: [],
      labelwidth: '120px',
      contentsTree: [],
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }]

      }
    }
  },
  created() {
    if (this.isEdit) {
      const fileName = this.$route.params.fileName
      this.getBookData(fileName)
    }
  },
  methods: {
    getBookData(fileName) {
      getBook(fileName).then(response => {
        this.setData(response.data)
      })
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text)
      }
    },
    setDefault() {
      // this.postForm = Object.assign({}, defaultForm)
      this.contentsTree = []
      this.fileList = []
      this.$refs.postForm.resetFields()
    },
    setData(data) {
      const {
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath
      } = data
      this.postForm = {
        ...this.postForm,
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath
      }
      this.contentsTree = contentsTree
      this.fileList = [{ name:originalName || fileName, url }]
    },
    onUploadSuccess(data) {
      console.log('onUploadSuccess', data)
      this.setData(data)
      // this.setData(data)
    },
    
    onUploadRemove() {
      this.setDefault()
    },

    submitForm() {
      const onSuccess =(response) => {
        const {msg} = Response
                this.$notify({
                  title: '操作成功',
                  message: msg,
                  type: 'success',
                  duration: 2000
                })
                this.loading = false
      }

      if(!this.loading){
        this.loading = true
        this.$refs.postForm.validate(( valid, fields )=> {
          console.log(valid, fields)
          if (valid) {
            const book = Object.assign({}, this.postForm)
            delete book.contentsTree
            // console.log(book)
            // 调用服务端接口
            if(!this.isEdit){
              createBook(book).then(response => {
                onSuccess(response)
                this.setDefault()
              }).catch(()=>{
                this.loading = false
              })
            } else {
              // 更新电子书
              updateBook(book).then(response => {
                onSuccess(response)
              }).catch(()=>{
                this.loading = false
              })
            }
          } else {
            const message = fields[Object.keys(fields)[0]][0].message
            this.$message({ message, type:'error'})
            this.loading = false
          }
        })
      }
    },

    showGuide() {
      console.log('showGuide**********')
    }
  }
}
</script>
<style lang="scss" scoped>
.detail-container {
  padding: 40px 45px 20px 50px;

  .preview-img {
    width: 200px;
    height: 270px;
  }

  .contents-wrapper {
    padding: 5px 0;
  }
}
</style>
