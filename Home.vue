<template>
  <section class="Home pt-4 sm:pt-6">
    <Alert class="pt-0 pb-2 sm:py-4">
      <AlertTitle class="font-bold hidden sm:flex sm:gap-2"> <RocketIcon class="h-4 w-4 hidden sm:flex" /> Heads up!</AlertTitle>
      <AlertDescription class="p-0 text-xs sm:text-sm">
        <p class="pt-2">无限图片储存数量，你可以上传不限数量的图片！</p>
        <p>图片首次访问后缓存，"永久"有效，包括全球分布的 CDN，以确保尽可能快地提供图像.</p>
        <p>FreeImg图床 是 <a class="text-slate-400" href="https://www.freeimg.cn" target="_blank" title="FreeImg官网">FreeImg官网</a> 支持并维护的文件上传项目，致力于为用户提供稳定的永久存储服务。</p>
        <!-- <p style="font-weight: bold">开源地址: <a class="text-[#0969da]" href="https://github.com/uxiaohan/ZYCS-IMG" target="_blank">FreeImg图床</a></p> -->
      </AlertDescription>
    </Alert>

    <!-- 参数输入栏 -->
    <div class="mb-4 flex flex-wrap gap-4 items-center">
      <label>Token: <input v-model="uploadToken" placeholder="请输入token（可复制粘贴）" style="width:300px" autocomplete="off" autocorrect="off" spellcheck="false" @paste.stop @contextmenu.stop /></label>
      <label>储存ID: <input v-model="storageId" placeholder="storage_id" style="width:80px" /></label>
      <label>相册ID: <input v-model="albumId" placeholder="album_id(可选)" style="width:80px" /></label>
      <button class="px-3 py-1 rounded bg-blue-500 text-white hover:bg-blue-600 transition" @click="saveParams">保存</button>
    </div>

    <!-- 工具栏 -->
    <div class="pt-6 flex items-center text-sm">
      <div class="sync shrink-0">
        <RadioGroup default-value="sync" class="flex items-center gap-4 [&>label]:flex [&>label]:items-center [&>label]:space-x-2 [&>label]:cursor-pointer">
          <Label for="sync">
            <RadioGroupItem id="sync" value="sync" />
            <span>Freeimg</span>
          </Label>
          <Label for="nosync">
            <RadioGroupItem id="nosync" value="nosync" disabled />
            <span class="text-gray-300">待定</span>
          </Label>
        </RadioGroup>
      </div>
    </div>
    <!-- 上传 -->
    <Upload v-model="fileList" :UploadConfig="UploadConfig" :uploadAPI="uploadAPI" :uploadToken="uploadToken" :storageId="storageId" :albumId="albumId" />
    <section v-show="fileList.length" class="vh-tools"><Button @click="fileList = []">清空</Button><Button @click="vh.CopyText(fileList.map((i: any) => i.upload_blob).join('\n'))">复制全部</Button></section>
    <!-- 展示 -->
    <ResList v-model="fileList" :nodeHost="nodeHost" />
  </section>
</template>
<script setup lang="ts">
import vh from 'vh-plugin';
import { ref, watch, onMounted } from 'vue';
import { formatURL } from '@/utils/index';
import { Button } from '@/components/ui/button';
import Upload from '@/components/Upload/Upload.vue';
import ResList from '@/components/ResList/ResList.vue';
import { RocketIcon } from '@radix-icons/vue';
import { Label } from '@/components/ui/label';
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
// IPFS节点
const nodeHost = ref<string>('https://www.freeimg.cn/api/v2/upload');
// 上传接口
const uploadAPI = ref<string>('https://www.freeimg.cn/api/v2/upload'); // 使用freeimg v2上传接口
// 上传配置
const UploadConfig = ref<any>({
  AcceptTypes: 'image/*', // 允许上传的类型，使用逗号分隔
  Max: 0, //多选个数，0为不限制
  MaxSize: 15, //单个文件大小限制，单位：MB
});
// 上传参数
const uploadToken = ref('');
const storageId = ref('3'); // 默认3
const albumId = ref('');

// 保存参数到localStorage
function saveParams() {
  localStorage.setItem('freeimg_upload_token', uploadToken.value ?? uploadToken);
  localStorage.setItem('freeimg_storage_id', storageId.value ?? storageId);
  localStorage.setItem('freeimg_album_id', albumId.value ?? albumId);
}
// 页面加载时自动回显参数
onMounted(() => {
  uploadToken.value = localStorage.getItem('freeimg_upload_token') || '';
  storageId.value = localStorage.getItem('freeimg_storage_id') || '3';
  albumId.value = localStorage.getItem('freeimg_album_id') || '';
});
// 上传列表
const fileList = ref<Array<any>>(JSON.parse(localStorage.getItem('zychUpImageList') || '[]'));
watch(fileList, (newVal) => {
  localStorage.setItem(
    'zychUpImageList',
    JSON.stringify(
      newVal
        .filter((i: any) => i.upload_status == 'success')
        .map((i: any) => {
          i.upload_blob = formatURL({ nodeHost: nodeHost.value }, i.upload_result);
          return i;
        }),
    ),
  );
});
</script>

<style scoped lang="less">
@import 'Home.less';
input, textarea {
  user-select: auto !important;
  -webkit-user-select: auto !important;
  -moz-user-select: auto !important;
  -ms-user-select: auto !important;
}
</style>
