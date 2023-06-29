<template>
  <div class="cdrag-container">
    <div class="left-box">
      <div class="item" v-for="(item, index) in materiaList" :key="index" :data-index="index" draggable="true">
        <img class="img" v-if="item.img" :data-index="index" :src="item.img" alt="" >
        <span class="text" v-if="item.text"  :style="{ fontSize: item.size + 'px', color: item.color}">{{ item.text }}</span>
      </div>
    </div>
    <canvas id="CDrag-canvas" ></canvas>
  </div>
</template>
<script setup>
import CDrag from 'cdrag'
import { onMounted, ref, onUnmounted } from 'vue';

let cDrag = null
let canvas = null
const materiaList = [{ text: '将我拖动到右侧画布上', size: 18, color: 'red' }, { img: 'https://images.pexels.com/photos/5326909/pexels-photo-5326909.jpeg?auto=compress&cs=tinysrgb&w=1600', width: 200, height: 200}]
let drawList = [
    {
      left: 200,
      top: 200,
      rotate: 45,
      width: 200,
      height: 160,
      zIndex: 0,
      img: 'https://images.pexels.com/photos/943905/pexels-photo-943905.jpeg?auto=compress&cs=tinysrgb&w=1600'
    },
    {
      left: 60,
      top: 60,
      rotate: 0,
      zIndex: 1,
      text: '引力波',
      size: 20,
      color: '#E6A23C'
    }
  ]

onMounted (() => {
  canvas = document.getElementById('CDrag-canvas')
  // 设置正确的width、height 可保证绘制分辨率不失真
  canvas.width = canvas.clientWidth
  canvas.height = canvas.clientHeight
  // CDrag 的配置参数
  const config = {
     // Canvas HTMLElement 强制性必填
    canvas, 
     // 要渲染的图形列表
    drawList,
    // 列表更新回调
    update: (newList) => { drawList = newList },
    // drawList数据项各字段name，可根据业务修改
    options: {
        left: 'left', // x轴距离
        top: 'top', // y轴距离
        rotate: 'rotate', // 旋转角度
        width: 'width', // 宽度（仅图片有效）
        height: 'height', // 高度（仅图片有效）
        zIndex: 'zIndex', // 渲染层级
        img: 'img', // 图片地址 （仅图片有效）
        text: 'text', // 文本内容 （仅文本有效）
        color: 'color', // 文本颜色 （仅文本有效）
        size: 'size', // 文本大小（仅文本有效）
      },
      // 图形选中时边框与控件颜色
      theme: '#396FFF',
      // 只展示不可操作(删除、变形、旋转)
      readOnly: false,
      // 画布是否可移动(仅在readOnly为true时有效)
      move: true,
      // 画布是否可缩放(仅在readOnly为true时有效)
      scale: true,
    }
  
  // 创建 PopupControl
  cDrag = new CDrag(config)
  onLeftDrag()
})

// 左侧素材拖动
function onLeftDrag ()  {
  document.querySelector('.left-box').addEventListener("dragend", (event) => {
  const { target: {  dataset: {index} }, clientX, clientY } = event
  let select = materiaList[index]
  if (select.img) {
    cDrag.addDraw({ ...select, left: clientX - (select.width / 2) - canvas.offsetLeft, top: clientY - (select.height / 2) - canvas.offsetTop })
  } else if (select.text) {
    const w = select.text.length * select.text.length
    cDrag.addDraw({ ...select, left: clientX - (w / 2) - canvas.offsetLeft, top: clientY - (select.size / 2) - canvas.offsetTop })
  }
})

}
onUnmounted(() => {
  cDrag.destroy()
})
</script>
<style>
.cdrag-container {
  user-select: none;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-flow: row nowrap;
  justify-content: flex-start;
  align-items: stretch;
}
.left-box {
  box-sizing: border-box;
  flex-basis: 20%;
  padding: 20px;
}
.item {
  height: 200px;
  margin-bottom: 20px;
  text-align: center;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
  border-radius: 4px;
  cursor: pointer;
}
.img {
  width: 100%;
  object-fit: cover;
}
.text {
  line-height: 200px;
}
#CDrag-canvas {
  cursor: pointer;
  flex: 1;
  border: 1px solid #000;
}
</style>