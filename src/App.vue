<template>
  <el-container>
    <el-header>
      <el-button-group>
        <el-button @click="triggerFileInput">读取图片</el-button>
        <el-button @click="handleEmbed">嵌入</el-button>
        <el-button @click="handleButtonClick('store')">存储图片</el-button>
        <el-button @click="handleButtonClick('extract')">提取</el-button>
      </el-button-group>
      <input type="file" ref="fileInput" style="display: none" @change="handleFileChange" accept=".bmp"/>
    </el-header>
    <el-main>
      <el-row :gutter="20">
        <!--原图显示区域-->
        <el-col :span="8">
          <el-card shadow="always" style="height: auto">
            <template #header>
              <div class="clearfix">
                <span>原图</span>
              </div>
            </template>
            <img v-if="imageSrc" :src="imageSrc" alt="原图" class="display-image"/>
          </el-card>
        </el-col>

        <el-col :span="8">

          <!--显示计算出来的可嵌入信息的最大长度-->
          <el-card shadow="always" style="height: 270px">
            <template #header>
              <div class="clearfix">
                <span>显示计算出来的可嵌入信息的最大长度</span>
              </div>
            </template>
            <!-- 显示最大长度 -->
          </el-card>


          <!--显示提取信息-->
          <el-card shadow="always" style="margin-top: 20px; height: auto;">
            <template #header>
              <div class="clearfix">
                <span>输入要嵌入的信息 / 显示提取出的信息</span>
              </div>
            </template>
            <el-input
                v-model="embedMessage"
                style="width: 100%"
                :autosize="{ minRows: 3, maxRows: 5 }"
                type="textarea"
                placeholder="请输入要嵌入的信息"
            />
            <div v-if="extractedMessage" class="extracted-message">
              <el-divider></el-divider>
              <p>提取到的信息:</p>
              <el-input
                  v-model="extractedMessage"
                  style="width: 100%"
                  type="textarea"
                  disabled
              />
            </div>
          </el-card>
        </el-col>
        <el-col :span="8">
          <el-card shadow="always" style="height: auto">
            <template #header>
              <div class="clearfix">
                <span>嵌入信息后的图</span>
              </div>
            </template>
            <!-- 嵌入后的图 !!直接嵌入 imageAfterEmbed-->
            <img v-if="imageAfterEmbed" :src="imageAfterEmbed" alt="嵌入信息后的图" class="display-image"/>
          </el-card>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      imageSrc: null,
      imageBase64: null,
      embedMessage: '',
      extractedMessage: '',
      imageAfterEmbed: null
    };
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file && file.type === 'image/bmp') {
        //使用FileReader对象将图片文件转换为DataURL（Base64编码的URL）
        const reader = new FileReader();
        reader.onload = (e) => {
          this.imageSrc = e.target.result;
          this.imageBase64 = e.target.result.split(',')[1];//提取DataURL的Base64部分
          this.uploadImage(this.imageBase64);//// 调用上传函数
        };
        reader.readAsDataURL(file);
      } else {
        this.$message.error('请选择一个 BMP 格式的图片');
      }
    },
    uploadImage(base64) {
      // 后端上传
      console.log('上传图片的 Base64 编码:', base64);
      // 使用 fetch 发送请求
      fetch('后端接口地址', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({image: base64})
      })
          .then(response => response.json())
          .then(data => {
            console.log('图片上传成功:', data);
          })
          .catch(error => {
            console.error('图片上传失败:', error);
          });
    },
    handleEmbed() {
      // 嵌入信息发送到后端
      const embedData = {
        image: this.imageBase64,
        message: this.embedMessage
      };
      console.log('嵌入的信息:', embedData);
      fetch('后端嵌入接口地址', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(embedData)
      })
          .then(response => response.json())
          .then(data => {
            console.log('信息嵌入成功:', data);
          })
          .catch(error => {
            console.error('信息嵌入失败:', error);
          });
    },
    handleButtonClick(action) {
      if (action === 'extract') {
        // 提取信息
        this.extractedMessage = '这是提取到的信息';
        fetch('后端提取接口地址', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({image: this.imageBase64})
        })
            .then(response => response.json())
            .then(data => {
              this.extractedMessage = data.extractedMessage;
            })
            .catch(error => {
              console.error('信息提取失败:', error);
            });
      }
      console.log(action);
    }
  }
};
</script>

<style>
.el-header {
  background-color: #409EFF;
  color: white;
  line-height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.el-main {
  padding: 20px;
}

.el-card {
  height: auto;
}

.display-image {
  max-height: calc(100vh - 200px); /* 设置最大高度，确保不超过页面高度 */
  width: auto;
  display: block;
  margin: 0 auto;
}

.extracted-message {
  margin-top: 20px;
}
</style>
