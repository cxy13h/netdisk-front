<template>
  <div>
    <!-- 文件上传 -->
    <el-upload
        :file-list="fileList"
        :auto-upload="true"
        :http-request="handleUpload"
        drag
    >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">拖拽文件到此处，或<em>点击上传</em></div>
    </el-upload>

    <!-- 上传进度 -->
    <div v-if="progress > 0">
      <el-progress :percentage="progress" status="success"></el-progress>
    </div>

    <!-- 文件信息 -->
    <div v-if="fileInfo">
      <p>文件名: {{ fileInfo.name }}</p>
      <p>文件大小: {{ (fileInfo.size / 1024 / 1024).toFixed(2) }} MB</p>
    </div>
  </div>
</template>

<script>
import SparkMD5 from "spark-md5";

export default {
  data() {
    return {
      fileList: [], // 文件列表
      fileInfo: null, // 当前选中的文件信息
      chunkSize: 2 * 1024 * 1024, // 分片大小 2MB
      totalChunks: 0, // 总分片数
      uploadedChunks: 0, // 已上传分片数
      progress: 0, // 上传进度百分比
      fileMd5: "", // 文件的 MD5 值
    };
  },
  methods: {
    /**
     * 自定义上传逻辑 (http-request)
     */
    async handleUpload(options) {
      const {file} = options;

      // 计算文件 MD5
      // this.$message.info("正在计算文件 MD5...");
      this.fileMd5 = await this.calculateFileMd5(file);
      console.log("文件 MD5:", this.fileMd5);

      // 初始化分片信息
      this.fileInfo = file;
      this.totalChunks = Math.ceil(file.size / this.chunkSize);
      this.uploadedChunks = 0;
      this.progress = 0;

      // this.$message.info("开始上传...");
      for (let chunkIndex = 0; chunkIndex < this.totalChunks; chunkIndex++) {
        const start = chunkIndex * this.chunkSize;
        const end = Math.min(file.size, start + this.chunkSize);
        const chunk = file.slice(start, end); // 获取分片

        const formData = new FormData();
        formData.append("file", chunk);
        formData.append("fileMd5", this.fileMd5);
        formData.append("chunkIndex", chunkIndex + 1); // 分片编号
        formData.append("totalChunks", this.totalChunks); // 总分片数

        try {
          // 发送分片到后端
          await fetch("/netdiskFile/uploadChunk", {
            method: "POST",
            body: formData,
          });

          this.uploadedChunks++; // 更新已上传分片数
          this.progress = Math.round((this.uploadedChunks / this.totalChunks) * 100);
        } catch (error) {
          // this.$message.error(`分片 ${chunkIndex + 1} 上传失败`);
          break; // 出现错误，停止上传
        }
      }

      if (this.uploadedChunks === this.totalChunks) {
        // this.$message.success("文件上传完成！");
      }
    },

    /**
     * 计算文件 MD5
     */
    async calculateFileMd5(file) {
      return new Promise((resolve, reject) => {
        const chunkSize = 2 * 1024 * 1024; // 每片 2MB
        const chunks = Math.ceil(file.size / chunkSize);
        const spark = new SparkMD5.ArrayBuffer();
        const fileReader = new FileReader();
        let currentChunk = 0;

        fileReader.onload = (e) => {
          spark.append(e.target.result); // 添加到 MD5 计算中
          currentChunk++;

          if (currentChunk < chunks) {
            loadNext(); // 继续读取下一片
          } else {
            resolve(spark.end()); // 返回最终的 MD5
          }
        };

        fileReader.onerror = () => reject("文件 MD5 计算失败");

        function loadNext() {
          const start = currentChunk * chunkSize;
          const end = Math.min(file.size, start + chunkSize);
          fileReader.readAsArrayBuffer(file.slice(start, end)); // 读取分片
        }

        loadNext(); // 开始读取第一片
      });
    },
  },
};
</script>

<style scoped>
.el-progress {
  margin-top: 20px;
}
</style>
