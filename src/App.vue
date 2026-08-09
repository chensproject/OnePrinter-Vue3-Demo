<template>
  <div class="main">
    <div>
      <h1>OnePrinter接口打印功能演示</h1>
    </div>
    <el-form :model="form" label-width="auto">
      <el-form-item label="人物列表">
        <el-checkbox-group v-model="form.names">
          <el-checkbox :label="item" :value="item" v-for="(item, index) in nameList" :key="index" />
        </el-checkbox-group>
      </el-form-item>
    </el-form>

    <div ref="pdfContent" class="card-container">
      <div v-if="form.names.length === 0" class="card">请选择人物</div>
      <div class="w-flex" style="flex-direction: column;" v-for="name in form.names" :key="name">
        <div class="card">{{ name }}</div>
        <el-button type="primary" @click="generatePDF(name)">下载文件</el-button>
      </div>
    </div>

    <div>
      <el-form :model="form" label-width="auto" style="width: 600px;">
        <el-form-item label="打印机" required>
          <el-select v-model="form.printer" placeholder="请选择打印机" @change="changePrinter">
            <el-option :value="item.name" v-for="(item, index) in printers" :key="index"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="纸张名称" required>
          <el-select v-model="form.paper_size" placeholder="请选择纸张">
            <el-option :value="item.name" v-for="(item, index) in paperSizeList" :key="index">
              <div class="w-flex w-gap-20">
                <div>{{ item.name }}</div>
                <div class="w-p-r-20">
                  {{ item.width }} × {{ item.height }} 毫米
                </div>
              </div>
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="打印份数" required>
          <el-input-number v-model="form.copies" controls controls-position="right" :min="1"
            style="width: 100%;"></el-input-number>
        </el-form-item>
      </el-form>
      <el-button type="primary" @click="toPrint" style="width: 100%;">打印文件</el-button>
    </div>

    <div style="margin-top: 60px;">
      请先下载OnePrinter服务端： <el-link href="https://docs.wangwei.ltd/oneprinter/"
        target="_blank">https://docs.wangwei.ltd/oneprinter/</el-link>
    </div>
    <div>
      本演示地址： <el-link href="https://demo.oneprinter.cn/" target="_blank">https://demo.oneprinter.cn/</el-link>
    </div>
    <div>
      本演示的gitee下载地址：<el-link href="https://gitee.com/chenrongbin/OnePrinter-Vue3-Demo"
        target="_blank">https://gitee.com/chenrongbin/OnePrinter-Vue3-Demo</el-link>
    </div>
    <div>
      本演示的github下载地址：<el-link href="https://github.com/chensproject/OnePrinter-Vue3-Demo"
        target="_blank">https://github.com/chensproject/OnePrinter-Vue3-Demo</el-link>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue';
import axios from 'axios';
import html2canvas from 'html2canvas';
import { ElMessage } from 'element-plus';

const form = ref({
  names: [],
  printer: '',
  paper_size: "",
  copies: 1,
});
const nameList = ref(['郭靖', '黄蓉', '杨过', '小龙女', '张无忌', '赵敏', '令狐冲', '任盈盈', '乔峰', '段誉']);

const api = "http://127.0.0.1:10081/";
// const api = "http://192.168.31.104:10081/"
// const api = "https://api.oneprinter.cn/api/"
const secret_key = ""  //使用外网接口时，需要安全码，请在OnePrinter界面中【我的账户】里获取


const printers = ref([]);
const getPrinter = async () => {
  try {
    let obj = {
      secret_key: secret_key
    }
    const response = await axios.post(api + 'getPrinter', obj);
    if (response.data.code === 1) {
      printers.value = response.data.data;
      ElMessage.success('打印机获取成功');
    } else {
      console.error(response.data.info);
      ElMessage.error(response.data.info || '获取打印机失败');
    }
  } catch (error) {
    console.error('请求失败:', error.message);
    ElMessage.error('网络异常，请稍后重试');
  }
};

// 根据打印机切换纸张
const paperSizeList = ref([])
const changePrinter = (row) => {
  form.value.paper_size = ""
  printers.value.forEach((item) => {
    if (item.name === form.value.printer) {
      paperSizeList.value = item.papers;
    }
  })
}

const pdfContent = ref(null);

// 下载卡片
const generatePDF = async (name) => {
  if (!name) {
    ElMessage.warning('请选择一个人物');
    return;
  }
  await nextTick();
  const container = pdfContent.value;
  if (!container) return;
  const cards = container.querySelectorAll('.card');
  let targetCard = null;
  for (let card of cards) {
    if (card.textContent.trim() === name) {
      targetCard = card;
      break;
    }
  }
  if (!targetCard) {
    ElMessage.error('未找到卡片');
    return;
  }

  const timestamp = Date.now();
  const fileName = `${name}_${timestamp}.png`; // 改为 PNG 后缀

  // 使用 html2canvas 截取卡片
  const canvas = await html2canvas(targetCard, {
    scale: 2,               // 高清输出
    useCORS: true,
    backgroundColor: '#ffffff'
  });
  // 生成图片数据（PNG 格式）
  const imgData = canvas.toDataURL('image/png');
  // 创建下载链接并自动点击
  const link = document.createElement('a');
  link.download = fileName;
  link.href = imgData;
  link.click();
  ElMessage.success(`「${name}」图片已下载`);
};

// 打印所有卡片-每张卡片调用一次打印接口
const toPrint = async () => {
  if (!form.value.printer) {
    ElMessage.warning('请先选择打印机');
    return;
  }
  if (form.value.names.length === 0) {
    ElMessage.warning('请至少选择一个人物');
    return;
  }
  await nextTick();
  if (!pdfContent.value) return;
  const cards = pdfContent.value.querySelectorAll('.card');
  if (cards.length === 0) {
    ElMessage.error('未找到卡片元素');
    return;
  }
  // 逐个卡片发送打印请求
  for (let i = 0; i < cards.length; i++) {
    const card = cards[i];
    const name = card.textContent.trim();
    try {
      // 生成卡片
      const canvas = await html2canvas(card, {
        scale: 2,
        useCORS: true,
        backgroundColor: '#ffffff'
      });
      const imgData = canvas.toDataURL('image/jpeg', 0.95);
      let obj = {
        printer: form.value.printer,
        paper_size: form.value.paper_size,
        copies: form.value.copies,
        base64: imgData,
        // url: "", // 使用外网接时，要把生成的文件存到自己的服务器上，然后传文件路径即可。
        secret_key: secret_key,
        // 使用【模板打印】时，请在OnePrinter服务商设置好打印模板，然后【下载字段】json文件，放在这里就可以了，不用传url或base64
        template: {
          code: "",
          data: [],
        }
      }
      const response = await axios.post(api + 'print', obj);
      if (response.data.code === 1) {
        ElMessage.success(`第 ${i + 1} 张卡片「${name}」打印成功`);
      } else {
        console.error(`第 ${i + 1} 张卡片打印失败:`, response.data.info);
        ElMessage.error(`「${name}」打印失败: ${response.data.info || '未知错误'}`);
      }
    } catch (error) {
      ElMessage.error(`「${name}」打印请求异常: ${error.message}`);
    }
  }
};

onMounted(() => {
  getPrinter()
});
</script>


<style scoped>
.main {
  margin: auto;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: auto;
  gap: 20px;
}

.w-flex {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

.card {
  width: 300px;
  height: 100px;
  background: rgb(198, 226, 255);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  box-sizing: border-box;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: flex-start;
}
</style>
