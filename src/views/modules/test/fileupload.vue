<template>
  <div>
      <el-button type="primary" @click="uploadHandle()">上传文件</el-button>
      <upload v-if="uploadVisible" ref="upload"></upload>
      <img :src="imageUrl">
  </div>
</template>

<script>
import Upload from './file-upload'
export default {
    data() {
        return {
            uploadVisible: false,
            imageUrl: this.$http.adornUrl("/upload/eb2f76ba-4391-4038-8971-ab61645a32d1.jpg")
        }
    },
    components: {
        Upload
    },
    methods: {
        uploadHandle () {
            this.uploadVisible = true
            this.$nextTick(() => {
            this.$refs.upload.init()
            })
        }
    }
}
</script>

<style>

</style>