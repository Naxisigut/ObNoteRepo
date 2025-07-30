# Uniapp 小程序自定义相机组件

## 功能介绍

这是一个功能完整的自定义相机页面，适用于微信小程序，支付宝小程序。主要功能包括：

- 📷 实时摄像头预览
- 🖼️ 支持拍照和从相册选择图片
- 📱 多张图片预览（最多4张）
- 🗑️ 图片删除功能
- ✨ 拍照闪光动画效果
- 🔙 自定义返回按钮（适配状态栏）

![自定义相机页面](attachments/custom-camera.png)

## 主要特性

### 1. 相机功能
- 后置摄像头拍照
- 高质量图片输出
- 拍照动画反馈

### 2. 图片管理
- 最多支持4张图片预览
- 点击预览区域可指定拍照位置
- 支持删除已拍摄的图片

### 3. 用户体验
- 拍照时的闪光动画
- 按钮缩放反馈
- 友好的错误提示

### 4. 平台适配
- 微信小程序端条件编译返回按钮（`#ifdef MP-WEIXIN`）
- 自动适配状态栏和标题栏高度
- **原因说明**：微信小程序在全屏相机页面中，系统默认的导航栏会被隐藏，用户无法通过系统返回按钮退出页面。因此需要自定义返回按钮，并使用条件编译确保只在微信小程序端显示，避免在其他平台（如H5、App端）出现冗余的返回按钮。同时需要适配不同设备的状态栏高度，确保返回按钮位置正确。

## 使用方法
一般作为独立页面使用。

## 代码实现

### 模板部分

```vue
<template>
  <view class="container">
    <!-- 摄像头预览区域 -->
    <view class="camera-container">
      <camera 
        class="camera"
        device-position="back"
        flash="off"
        @error="onCameraError"
        id="camera"
      ></camera>
      <!-- 返回上一页 -->

      <!-- 拍照闪光动画遮罩 -->
      <view class="flash-overlay" :class="{ flashing: isFlashing }"></view>
      
      <!-- 手包图标和角标 -->
      <view class="overlay">
        <!-- 手包图标 -->
        <view class="handbag-icon">
          <image src="/static/camera.png" mode="aspectFit" class="handbag-image"></image>
        </view>
        
        <!-- L形角标 -->
        <view class="corner-brackets">
          <view class="corner top-left"></view>
          <view class="corner bottom-right"></view>
        </view>
      </view>
    </view>
    
    <!-- 图片预览区域 -->
    <view class="preview-container">
      <view class="preview-row">
        <view 
          v-for="(item, index) in previewImages" 
          :key="index"
          class="preview-item"
          :class="{ active: selectedIndex === index }"
          @click="selectPreviewItem(index)"
        >
          <view class="preview-placeholder" :class="{ empty: !item.src }">
            <image v-if="item.src" :src="item.src" mode="aspectFill" class="preview-image" />
            <!-- 删除按钮 -->
            <view v-if="item.src" class="delete-btn" @click.stop="deleteImage(index)">
              <text class="delete-icon">×</text>
            </view>
          </view>
        </view>
      </view>
    </view>
    
    <!-- 底部控制按钮 -->
    <view class="bottom-controls">
      <view class="control-button album-btn" @click="chooseFromAlbum">
        <text class="btn-text">从相册选择</text>
      </view>
      <view class="camera-button" :class="{ taking: isTaking }" @click="takePhoto">
        <view class="camera-btn-inner"></view>
      </view>
      <view class="control-button confirm-btn" @click="confirmUse">
        <text class="btn-text">确认使用</text>
      </view>
    </view>
  </view>
</template>
```

### script - 核心

```javascript
// 预览图片数组
const selectedIndex = ref(-1) // 当前选中的预览区域索引
const previewImages = reactive([
  { src: '', selected: false },
  { src: '', selected: false },
  { src: '', selected: false },
  { src: '', selected: false }
])
const selectPreviewItem = (index) => selectedIndex.value = index
// 删除图片
const deleteImage = (index) => {
  previewImages[index].src = ''
  // 如果删除的是当前选中的区域，取消选中状态
  if (selectedIndex.value === index) {
    selectedIndex.value = -1
  }
}
// 拍照功能
const takePhoto = () => {
  if (isTaking.value) return
  const hasEmptySlot = selectedIndex.value >= 0 || previewImages.some(item => !item.src)
  if (!hasEmptySlot) {
    uni.showToast({
      title: '请先删除当前已拍摄的图片',
      icon: 'none'
    })
    return
  }
  playTakePhotoAnimation()
  
  const ctx = uni.createCameraContext('camera')
  console.log('ctx', ctx);
  ctx.takePhoto({
    quality: 'high',
    success: (res) => {
      console.log('takePhoto success', res);
      const imagePath = res.tempImagePath
      
      // 如果有选中的预览区域，直接填入
      if (selectedIndex.value >= 0) {
        previewImages[selectedIndex.value].src = imagePath
        // 拍照后取消选中状态
        selectedIndex.value = -1
      } else {
        // 如果没有选中，按顺序填入第一个空的位置
        const emptyIndex = previewImages.findIndex(item => !item.src)
        if (emptyIndex >= 0) {
          previewImages[emptyIndex].src = imagePath
        }
      }
    },
    fail: (err) => {
      console.error('拍照失败:', err)
      uni.showToast({
        title: '拍照失败',
        icon: 'none'
      })
    }
  })
}
// 从相册选择
const chooseFromAlbum = () => {
  // 检查是否有空位置
  const hasEmptySlot = selectedIndex.value >= 0 || previewImages.some(item => !item.src)
  if (!hasEmptySlot) {
    uni.showToast({
      title: '请先删除当前已拍摄的图片',
      icon: 'none'
    })
    return
  }
  
  uni.chooseImage({
    count: 1,
    sizeType: ['original', 'compressed'],
    sourceType: ['album'],
    success: (res) => {
      const imagePath = res.tempFilePaths[0]
      
      // 如果有选中的预览区域，直接填入
      if (selectedIndex.value >= 0) {
        previewImages[selectedIndex.value].src = imagePath
        selectedIndex.value = -1
      } else {
        // 如果没有选中，按顺序填入第一个空的位置
        const emptyIndex = previewImages.findIndex(item => !item.src)
        if (emptyIndex >= 0) {
          previewImages[emptyIndex].src = imagePath
        }
      }
    },
    fail: (err) => {
      console.error('选择图片失败:', err)
    }
  })
}



// 摄像头错误处理
const onCameraError = (e) => {
  console.error('摄像头错误:', e)
  uni.showToast({
    title: '摄像头启动失败',
    icon: 'none'
  })
}

// 确认使用
const confirmUse = () => {
  const images = previewImages.filter(item => item.src).map(item => item.src)
  if (images.length === 0) {
    uni.showToast({
      title: '请先拍摄图片',
      icon: 'none'
    })
    return
  }
  
  console.log('确认使用的图片:', images)
  // 这里可以添加后续处理逻辑
  uni.showToast({
    title: `已选择${images.length}张图片`,
    icon: 'success'
  })
}
```

### script - 动画

```javascript
//#region 拍照动画
// 拍照动画
const isFlashing = ref(false)
const isTaking = ref(false)
const playTakePhotoAnimation = () => {
  isTaking.value = true // 按钮缩放动画
  isFlashing.value = true // 闪光动画
  // 播放快门声音
  // uni.createInnerAudioContext() 可以用来播放声音

  setTimeout(() => {
    isFlashing.value = false
    isTaking.value = false
  }, 200)
}
//#endregion
```

### 返回上一页
```html
<!-- #ifdef MP-WEIXIN -->
<view class="back-btn">
  <view :style="{ height: statusBarHeight + 'px' }"></view>
  <view class="title" :style="{ height: titleBarHeight + 'px' }">
    <u-icon name="arrow-left" :size="20" color="#303133" @click.stop="goBack" class="back-icon"></u-icon>
  </view>
</view>
<!-- #endif -->

```

```js
import { getTitleBarHeight, getStatusBarHeight } from '@/utils/condition';
//#region 返回上一页
const statusBarHeight = getStatusBarHeight()
const titleBarHeight = getTitleBarHeight()
const goBack = () => uni.navigateBack()
//#endregion

```
## 注意事项
1. **权限配置**：需要在 `manifest.json` 中配置摄像头权限
2. **平台兼容**：微信小程序 支付宝小程序