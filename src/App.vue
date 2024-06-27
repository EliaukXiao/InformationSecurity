<template>
  <el-container>
    <el-header>
      <el-button-group>
        <el-button @click="triggerFileInput">读取图片</el-button>
        <el-button @click="handleEmbed">嵌入</el-button>
        <el-button @click="handleAddNoise">嵌入后加噪声</el-button>
        <el-button @click="handleCalculateAccuracy">计算比特准确率</el-button>
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
          <el-card shadow="always" style="height: 150px">
            <template #header>
              <div class="clearfix">
                <span>最大可嵌入信息长度: {{ maxEmbedLength }} 字节</span>
              </div>
            </template>
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
                class="embed-input"
                :maxlength="maxEmbedLength"
                :show-word-limit="true"
                type="textarea"
                placeholder="请输入要嵌入的信息"
            />
            <div v-if="extractedMessage" class="extracted-message">
              <el-divider></el-divider>
              <p>提取到的信息:</p>
              <el-input
                  v-model="extractedMessage"
                  class="embed-input"
                  type="textarea"
                  disabled
              />
            </div>
          </el-card>
          
          <!--显示准确率-->
          <el-card shadow="always" style="margin-top: 20px; height: auto;" v-if="accuracy !== null">
            <template #header>
              <div class="clearfix">
                <span>比特准确率</span>
              </div>
            </template>
            <p>{{ accuracy }}%</p>
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
      imageAfterEmbed: null,
      maxEmbedLength: 0, // 最大可嵌入长度
      accuracy: null // 准确率
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
          this.uploadImage(this.imageBase64); // 调用上传函数
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
      fetch('http://localhost:8081/upload', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({image: base64})
      })
          .then(response => response.json())
          .then(data => {
            console.log('图片上传成功:', data);
            this.maxEmbedLength = data.maxEmbedLength; // 设置最大可嵌入长度
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
      fetch('http://localhost:8081/embed', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(embedData)
      })
          .then(response => response.json())
          .then(data => {
            console.log('信息嵌入成功:', data);
            this.imageAfterEmbed = 'data:image/bmp;base64,' + data.image; // 更新嵌入后的图片
          })
          .catch(error => {
            console.error('信息嵌入失败:', error);
          });
    },
    handleAddNoise() {
      // 发送请求到后端，添加噪声
      const noiseData = {
        image: this.imageBase64,
        message: this.embedMessage
      };
      fetch('http://localhost:8081/addNoise', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(noiseData)
      })
          .then(response => response.json())
          .then(data => {
            console.log('加噪声成功:', data);
            this.imageAfterEmbed = 'data:image/bmp;base64,' + data.image; // 更新嵌入后加噪声的图片
          })
          .catch(error => {
            console.error('加噪声失败:', error);
          });
    },
    handleCalculateAccuracy() {
      // 计算比特准确率
      const accuracyData = {
        originalMessage: this.embedMessage,
        extractedMessage: this.extractedMessage
      };
      fetch('http://localhost:8081/calculateAccuracy', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(accuracyData)
      })
          .then(response => response.json())
          .then(data => {
            console.log('计算准确率成功:', data);
            this.accuracy = data.accuracy; // 更新准确率
          })
          .catch(error => {
            console.error('计算准确率失败:', error);
          });
    },
    handleButtonClick(action) {
      if (action === 'extract') {
        // 提取信息
        fetch('http://localhost:8081/extract', {
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
      } else if (action === 'store') {
        this.storeImage();
      }
      console.log(action);
    },
    storeImage() {
      const link = document.createElement('a');
      link.href = this.imageAfterEmbed;
      link.download = 'embedded_image.bmp';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }
  },
  watch: {
    embedMessage(newVal) {
      let byteLength = new TextEncoder().encode(newVal).length;
      while (byteLength > this.maxEmbedLength) {
        newVal = newVal.slice(0, -1); // 去掉最后一个字符
        byteLength = new TextEncoder().encode(newVal).length;
      }
      this.embedMessage = newVal;
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
  max-width: 100%; /* 设置最大宽度，确保图片宽度不超过父容器宽度 */
  max-height: 512px; /* 设置最大高度，确保图片高度不超过512px */
  display: block;
  margin: 0 auto;
}

.embed-input {
  height: 150px; /* 设置嵌入信息输入框的固定高度 */
}

.extracted-message {
  margin-top: 20px;
  height: 150px; /* 设置提取信息显示框的固定高度 */
}
</style>
