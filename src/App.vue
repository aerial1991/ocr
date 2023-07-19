<template>
  <div>
    <div>
      <a-input type="file" accept="image/*" @change="handleFileChange" placeholder="Basic usage" />
    </div>
    <a-divider />
    <div>
      <a-button type="primary" @click="uploadImage" block>上传图片</a-button>
    </div>
    <a-divider />
    <div>
      <a :href="downloadUrl" :download="fileName" v-if="downloadUrl">下载excel</a>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import axios from "axios"; // 引入axios库

export default {
  setup() {
    const file = ref(null); // 定义一个响应式变量存储文件对象
    const downloadUrl = ref(null); // 定义一个响应式变量存储下载链接
    const fileName = ref(null); // 定义一个响应式变量存储文件名

    // 定义一个函数处理文件选择事件
    const handleFileChange = (event) => {
      file.value = event.target.files[0]; // 获取选择的文件对象
      fileName.value = file.value.name.replace(/\.\w+$/, ".xlsx"); // 根据文件名生成excel文件名
    };

    // 定义一个函数上传图片
    const uploadImage = async () => {
      if (!file.value) return; // 如果没有选择文件则返回
      const reader = new FileReader(); // 创建一个文件读取器对象
      reader.readAsArrayBuffer(file.value); // 以数组缓冲区的方式读取文件
      reader.onload = async () => {
        const buffer = reader.result; // 获取读取结果
        console.log(buffer); // 打印二进制流到控制台
        try {
          const response = await axios.post(
            "https://api.textin.com/ai/service/v2/recognize/table?excel=1", // 请求地址加上URL参数
            buffer, // 请求体为二进制流
            {
              headers: {
                "Content-Type": "application/octet-stream", // 设置请求头的内容类型为二进制流
                "x-ti-app-id": "d9827d636c761d22059df3abd0522803", // 设置请求头的x-ti-app-id
                "x-ti-secret-code": "21345b46a19b9acd734dfd0d272cc1a0", // 设置请求头的x-ti-secret-code
              },
            }
          ); // 发送post请求并获取响应
          if (response.data.code === 200) {
            // 如果响应码为200则表示成功
            const excel = response.data.result.excel; // 获取响应中的excel文件的base64格式
            downloadUrl.value = `data:application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;base64,${excel}`; // 根据base64格式生成下载链接
          } else {
            // 否则表示失败
            alert(`请求失败，错误码：${response.data.code}`); // 弹出提示框显示错误码
          }
        } catch (error) {
          // 如果发生异常则捕获错误
          console.error(error); // 打印错误到控制台
          alert("请求失败，请检查网络或接口文档"); // 弹出提示框显示失败信息
        }
      };
    };

    return {
      file,
      downloadUrl,
      fileName,
      handleFileChange,
      uploadImage,
    };
  },
};
</script>
