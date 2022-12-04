<!-- staori生成pdf 示例 
总结：可用，但是存在问题：（2022年12月04日 satori-V0.0.44）
1. 适用于简单场景，非所见即所得，复杂场景可能问题比较多
2. 过程繁琐，需要简化提升效率、减少代码体积
3. 如官网所说只支持大部分的html和css
4. 更适应于react，v-staori做了层vue转react-like的工作，但维护是个问题
-->
<script setup lang="ts">
import { createApp, h, ref } from "vue";
import { renderToString } from "vue/server-renderer";
// 只用html方法就行，可替代
import { html as _html } from "satori-html";
// web必须使用wasm
// @ts-ignore
import satori, {init} from "satori/wasm";
// web必须使用yoga-wasm-web初始化wasm并传递实例给satori
import initYoga from 'yoga-wasm-web'
// 使用pdfkit将svg转pdf
// @ts-ignore
import PDFDocument from 'pdfkit/js/pdfkit.standalone.js'
import SVGtoPDF from 'svg-to-pdfkit'
// blob-stream使用blob实例了类似Node的Stream模块功能，内部需要Node的pollify，webpack5或vite这些pollify需要显示引入（详见vite.config.js）
// @ts-ignore
import * as blobStream from 'blob-stream'

import PDF from "./pdf.vue";

initSatori()

const title = ref('baidu')
const website = ref('www.baidu.com')

// 初始化satori
async function initSatori() {
  // wasm 从yoga-wasm-web/dist/yoga.wasm获取，然后放到public目录下或者cdn上
  const yoga = await initYoga(await fetch('/yoga.wasm').then(res => res.arrayBuffer()))
  // wasm 实例赋值给satori
  init(yoga)
}

// 点击生成pdf
async function onExportPdfClick() {
  // 需要至少一种字体
  const fontData = await fetch('https://unpkg.com/@fontsource/inter@4.5.2/files/inter-latin-ext-400-normal.woff')
  // 转buffer
  const fontData2Buffer = await fontData.arrayBuffer()
  
  // 1. vue 转 VNode（类react结构，都是类似的AST结构）
  const jsx = await html(PDF, {
    title: title.value,
    website: website.value,
  })
  console.log('vue-2-jsx', jsx);

  // 2. vnode 转 svg
  const svg = await satori(jsx, {
    // props: {
    //   title: 'OG Image Generator using Nuxt and Satori',
    //   website: 'v-satori.vercel.app',
    // },
    width: 1000,
    height: 627,
    fonts: [{ // 字体是必须的
      name: 'Inter',
      data: fontData2Buffer,
      weight: 400,
      style: 'normal',
    }]
  })
  console.log('jsx-2-svg', svg)

  // 3. svg 转 pdf （pdfkit）
  const doc = new PDFDocument({compress: false});
  SVGtoPDF(doc, svg, 0, 0);
  console.log('svg-2-pdf-doc', doc);

  // 4. pdfkit输出pdf流
  const stream = doc.pipe(blobStream());
  console.log('doc-2-stream', stream);
  
  // 5. 读流并转blob格式
  const pdfBlob: any = await new Promise((resolve) => {
    stream.on('finish', () => {
      const blob = stream.toBlob('application/pdf');
      resolve(blob)
    });
    doc.end()
  })
  console.log('pdf-2-bolb', pdfBlob)

  // 6. blob转ObjectURL并下载
  const a = document.createElement('a');
  a.href = URL.createObjectURL(pdfBlob);
  a.download = 'outFileName.pdf';
  a.click();  
}

// vue sfc -> VNode（从v-staori copy的）
async function html(component: any, props: any) {
  let Root;
  if (props)
    Root = createApp(h(component, props));
  else
    Root = createApp(component);
  const strComponent = await renderToString(Root);
  return _html(strComponent);
}
</script>

<template>
  <PDF :title="title" :website="website"/>
  <button>导出图片</button>
  <button @click="onExportPdfClick">导出PDF</button>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
