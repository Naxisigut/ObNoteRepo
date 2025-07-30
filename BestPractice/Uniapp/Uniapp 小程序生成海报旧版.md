本文是`Uniapp 小程序生成海报.md`的参考，不同处在于使用的是canvas的旧版api。当前微信小程序可以支持。

标签： #Uniapp #Canvas #海报生成 #小程序 #最佳实践 #跨端兼容 #Vue3 #图片处理 #文本绘制

## 页面
```json
{
  "path": "pages/poster/poster",
  "style": {
    "navigationBarTitleText" : "生成海报"
  }
}
```

```vue
<template>
  <m-canvas ref="myCanvasRef" :width="470" :height="690" />
  <button @click="createPoster">生成海报</button>
</template>

<script setup>
  import { ref } from 'vue'
  import MCanvas from './components/canvas.vue';
  const myCanvasRef = ref(null)
  function createPoster() {
    const test = {
      type: 'text',
      content: 'test',
      fontSize: 24,
      left: 'center',
      color: '#333',
      top: 100
    }
    // 背景图
    const bgEl = {
      type: 'image',
      url: 'https://file-recycle-test.kuarke.com/data/product/1/20250716181614/detail/20250716181614280_62443.png',
      left: 0,
      top: 0,
      width: 470,
      height: 690
    }
    // 小程序码白色背景
    const whiteBgEl = {
      type: 'block',
      color: '#fff',
      radius: 30, // 30px圆角
      left: 'center',
      top: 275,
      width: 245,
      height: 245
    }
    // 长按扫码 > 浏览臻品 > 获取权益
    const titelEl = {
      type: 'text',
      content: '长按扫码 > 浏览臻品 > 获取权益',
      color: '#333',
      fontSize: 20,
      left: 'center',
      top: 240
    }
    // 小程序码
    const mpCodeEl = {
      type: 'image',
      url: '/static/camera.png',
      left: 'center',
      top: titelEl.top + titelEl.fontSize + 50,
      width: 180,
      height: 180
    }
    // 头像
    const avatarEl = {
      type: 'image',
      url: 'https://file-recycle-test.kuarke.com/data/product/1/20250721115857/detail/20250721115857375_64573.png',
      radius: '50%',
      left: 'center',
      top: whiteBgEl.top + whiteBgEl.height + 55,
      width: 50,
      height: 50
    }
    // 昵称
    const nicknameEl = {
      type: 'text',
      content: 'Jerry',
      color: '#333',
      fontSize: 20,
      left: 'center',
      top: avatarEl.top + avatarEl.height + 30
    }
    // 配置项
    const options = [
      // test
      bgEl, 
      whiteBgEl, 
      titelEl, 
      mpCodeEl, 
      avatarEl, 
      nicknameEl
    ]
    // 调用myCanvas的onDraw方法，绘制并保存
    // myCanvasRef.value.testCanvasText()
    myCanvasRef.value.onDraw(options, (url) => {
      console.log(url)
    })
  }
</script>

<style lang="scss" scoped>
button{
  position: fixed;
  bottom: 0;

}
</style>

```

## 组件
```vue
<template>
  <canvas class="myCanvas" canvas-id="myCanvas" />
</template>

<script setup>
import { getCurrentInstance } from 'vue'
import { createPoster } from './handleCanvas.js'
const { proxy } = getCurrentInstance()
const props = defineProps({
  width: {
    type: Number,
    required: true
  },
  height: {
    type: Number,
    required: true
  }
})


// 导出方法给父组件用
defineExpose({
  onDraw(options, callback) {
    createPoster.call(
      // 当前上下文
      proxy,
      // canvas相关信息
      {
        id: 'myCanvas',
        width: props.width,
        height: props.height
      },
      // 元素集合
      options,
      // 回调函数
      callback
    )
  },
})
</script>

<style lang="scss" scoped>
  // 隐藏canvas
  .myCanvas {
    // left: -9999px;
    // bottom: -9999px;
    top: 0;
    position: fixed;
    // canvas宽度
    width: calc(1px * v-bind(width));
    // canvas高度
    height: calc(1px * v-bind(height));
  }
</style>

```

## 处理逻辑
```js
/** @生成海报 **/
export function createPoster(canvasInfo, options, callback) {
  uni.showLoading({
    title: '海报生成中…',
    mask: true
  })
  const myCanvas = uni.createCanvasContext(canvasInfo.id, this)
  var index = 0
  drawCanvas(myCanvas, canvasInfo, options, index, () => {
    myCanvas.draw(true, () => {
      // 延迟，等canvas画完
      const timer = setTimeout(() => {
        console.log(22222);
        savePoster.call(this, canvasInfo.id, callback)
        clearTimeout(timer)
      }, 1000)
    })
  })
}

// 绘制中
async function drawCanvas(myCanvas, canvasInfo, options, index, drawComplete) {
  let item = options[index]
  console.log(111, item);
  // 最大行数：maxLine  字体大小：fontSize  行高：lineHeight
  //    类型    颜色  left  right  top  bottom   宽      高     圆角    图片  文本内容
  let { type, color, left, right, top, bottom, width, height, radius, url, content, fontSize } = item
  radius = radius || 0
  const { width: canvasWidth, height: canvasHeight } = canvasInfo
  switch (type) {
    /** @文本 **/
    case 'text':
      if (!content) break
      // 根据字体大小计算出宽度
      myCanvas.setFontSize(fontSize)
      // 内容宽度：传了宽度就去宽度，否则取字体本身宽度
      item.width = width || myCanvas.measureText(content).width
      // left位置
      if (right !== undefined) {
        item.left = canvasWidth - right - item.width
      } else if (left === 'center') {
        item.left = canvasWidth / 2 - item.width / 2
      }

      // top位置
      if (bottom !== undefined) {
        item.top = canvasHeight - bottom - fontSize
      }
      
      drawText(myCanvas, item)
      break
    /** @图片 **/
    case 'image':
      if (!url) break
      var imageTempPath = await getImageTempPath(url)
      // left位置
      if (right !== undefined) {
        left = canvasWidth - right - width
      } else if (left === 'center') {
        left = canvasWidth / 2 - width / 2
      }

      // top位置
      if (bottom !== undefined) {
        top = canvasHeight - bottom - height
      }

      // 带圆角
      if (radius) {
        myCanvas.save()
        myCanvas.beginPath()
        // 圆形图片
        if (radius === '50%') {
          myCanvas.arc(left + width / 2, top + height / 2, width / 2, 0, Math.PI * 2, false)
        } else {
          if (width < 2 * radius) radius = width / 2
          if (height < 2 * radius) radius = height / 2
          myCanvas.beginPath()
          myCanvas.moveTo(left + radius, top)
          myCanvas.arcTo(left + width, top, left + width, top + height, radius)
          myCanvas.arcTo(left + width, top + height, left, top + height, radius)
          myCanvas.arcTo(left, top + height, left, top, radius)
          myCanvas.arcTo(left, top, left + width, top, radius)
          myCanvas.closePath()
        }
        myCanvas.clip()
      }
      myCanvas.drawImage(imageTempPath, left, top, width, height)
      myCanvas.restore()

      break
    /** @盒子 **/
    case 'block':
      // left位置
      if (right !== undefined) {
        left = canvasWidth - right - width
      } else if (left === 'center') {
        left = canvasWidth / 2 - width / 2
      }

      // top位置
      if (bottom !== undefined) {
        top = canvasHeight - bottom - height
      }
      if (width < 2 * radius) {
        radius = width / 2
      }
      if (height < 2 * radius) {
        radius = height / 2
      }
      myCanvas.beginPath()
      myCanvas.fillStyle = color
      myCanvas.strokeStyle = color
      myCanvas.moveTo(left + radius, top)
      myCanvas.arcTo(left + width, top, left + width, top + height, radius)
      myCanvas.arcTo(left + width, top + height, left, top + height, radius)
      myCanvas.arcTo(left, top + height, left, top, radius)
      myCanvas.arcTo(left, top, left + width, top, radius)
      myCanvas.stroke()
      myCanvas.fill()
      myCanvas.closePath()
      break
    /** @边框 **/
    case 'border':
      // left位置
      if (right !== undefined) {
        left = canvasWidth - right - width
      }

      // top位置
      if (bottom !== undefined) {
        top = canvasHeight - bottom - height
      }
      myCanvas.beginPath()
      myCanvas.moveTo(left, top)
      myCanvas.lineTo(left + width, top + height)
      myCanvas.strokeStyle = color
      myCanvas.lineWidth = width
      myCanvas.stroke()
      break
  }

  // 递归边解析图片边画
  if (index === options.length - 1) {
    drawComplete()
  } else {
    index++
    drawCanvas(myCanvas, canvasInfo, options, index, drawComplete)
  }
}

// 下载并保存
function savePoster(canvasId, callback) {
  uni.showLoading({
    title: '保存中…',
    mask: true
  })
  uni.canvasToTempFilePath(
    {
      canvasId,
      success(res) {
        console.log(33333);
        callback && callback(res.tempFilePath)
        uni.saveImageToPhotosAlbum({
          filePath: res.tempFilePath,
          success() {
            uni.showToast({
              icon: 'success',
              title: '保存成功！'
            })
          },
          fail() {
            uni.showToast({
              icon: 'none',
              title: '保存失败，请稍后再试~'
            })
          },
          complete() {
            uni.hideLoading()
          }
        })
      },
      fail(res) {
        console.log('图片保存失败：', res.errMsg)
        uni.showToast({
          icon: 'none',
          title: '保存失败，请稍后再试~'
        })
      }
    },
    this
  )
}

// 绘制文字（带换行超出省略…功能）
function drawText(ctx, item) {
  let { content, width, maxLine, left, top, lineHeight, color, fontSize } = item
  content = String(content)
  lineHeight = (lineHeight || 1.3) * fontSize

  // 字体
  ctx.setFontSize(fontSize)
  // 颜色
  ctx.setFillStyle(color)
  // 文本处理
  let strArr = content.split('')
  let row = []
  let temp = ''
  for (let i = 0; i < strArr.length; i++) {
    if (ctx.measureText(temp).width < width) {
      temp += strArr[i]
    } else {
      i-- //这里添加了i-- 是为了防止字符丢失，效果图中有对比
      row.push(temp)
      temp = ''
    }
  }
  row.push(temp) // row有多少项则就有多少行

  //如果数组长度大于2，现在只需要显示两行则只截取前两项,把第二行结尾设置成'...'
  if (row.length > maxLine) {
    let rowCut = row.slice(0, maxLine)
    let rowPart = rowCut[1]
    let text = ''
    let empty = []
    for (let i = 0; i < rowPart.length; i++) {
      if (ctx.measureText(text).width < width) {
        text += rowPart[i]
      } else {
        break
      }
    }
    empty.push(text)
    let group = empty[0] + '...' //这里只显示两行，超出的用...表示
    rowCut.splice(1, 1, group)
    row = rowCut
  }
  // 把文本绘制到画布中
  for (let i = 0; i < row.length; i++) {
    // 一次渲染一行
    ctx.fillText(row[i], left, top + i * lineHeight, width)
  }
}

// 获取图片信息
function getImageTempPath(url) {
  return new Promise((resolve) => {
    // 网络图片
    if (url.includes('http')) {
      uni.downloadFile({
        url,
        success: (res) => {
          console.log('downloadFile', res);
          uni.getImageInfo({
            src: res.tempFilePath,
            success: ({ path }) => resolve(path)
          })
        },
        fail: (res) => {
          console.log('downloadFile fail', res);
          uni.showToast({
            icon: 'none',
            title: url + res.errMsg
          })
        }
      })
      return
    }
    
    // base64图片
    // base64需要转为本地图片，canvas真机上不支持解析base64
    if (url.includes('base64')) {
      base64Save(url).then((src) => {
        uni.getImageInfo({
          src,
          success: ({ path }) => resolve(path)
        })
      })
    }
    
    // 本地图片
    resolve(url)
  })
}

function base64Save(base64File) {
  //base64File 需要加前缀
  const fsm = uni.getFileSystemManager() //获取全局文件管理器
  let extName = base64File.match(/data:\S+\/(\S+);/)
  if (extName) {
    //获取文件后缀
    extName = extName[1]
  }
  //获取自1970到现在的毫秒 + 文件后缀 生成文件名
  let fileName = Date.now() + '.' + extName
  return new Promise((resolve, reject) => {
    //写入文件的路径
    const filePath = uni.env.USER_DATA_PATH + '/' + fileName
    fsm.writeFile({
      filePath,
      data: base64File.replace(/^data:\S+\/\S+;base64,/, ''), //替换前缀为空
      encoding: 'base64',
      success: () => {
        resolve(filePath)
      },
      fail() {
        reject('写入失败')
      }
    })
  })
}

// 用于计算字符串实际长度，中文单个字符长度为1，英文和数字单个字符长度为0.5
export const getStrLength = (data) => {
  data = String(data)
  let length = 0
  for (let i = 0; i < data.length; i++) {
    length += data.charCodeAt(i) > 128 ? 1 : 0.5
  }
  return length
}

```