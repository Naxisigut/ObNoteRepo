# Uniapp scroll-view 横向滚动实现

标签： #Uniapp #scroll-view #横向滚动 #小程序 #最佳实践 #跨端兼容 #CSS #布局

## 需求说明
实现横向滚动列表，要求：
- 支持左右滑动浏览内容
- 左右两端与容器边缘保持适当间距
- 兼容微信小程序和支付宝小程序

效果预览：
![scroll-view-1](attachments/scroll-view-1.png)
![scroll-view-2](attachments/scroll-view-2.png)

## 推荐方案
**适用场景**：跨端兼容，稳定可靠

```html
<scroll-view scroll-x id="scroll-view">
  <view class="scroll-view-item-wrapper">
    <view v-for="item in 20" :key="item" class="scroll-view-item">
      {{ item }}
    </view>
  </view>
</scroll-view>
```

```scss
#scroll-view{
  width: 100%;
  box-sizing: border-box;
  .scroll-view-item-wrapper{
    // 关键代码，使得宽度=item总宽=scrollview宽度，最右的padding可以正常展示
    // 不能使用fit-content,在安卓支付宝环境下不生效;max-content兼容性更佳
    width: max-content; 
    box-sizing: border-box;
    display: flex;
    gap: 30rpx;
    padding: 16rpx 30rpx; // 左右padding为30rpx，滚动到两边时用padding隔开边缘
    background: burlywood;
  }
  .scroll-view-item{
    padding: 0 24rpx;
    font-size: 24rpx;
    line-height: 100rpx;
    border: 1rpx solid #E5E5E5;
    width: 200rpx; // 可以不指定，由内容自动撑开；指定时也不会被flex压缩
    // 防止安卓支付宝环境下文字被挤成一列
    flex-shrink: 0; 
    text-wrap: nowrap;
  }
}
```

**核心要点**：
- 使用内层包裹容器 `scroll-view-item-wrapper`
- 设置 `width: fit-content` 确保右侧 padding 正常显示
- 在双端均运作良好

## 备选方案
**适用场景**：仅支付宝小程序，不推荐使用

```html
<scroll-view scroll-x id="scroll-view">
  <view v-for="item in 20" :key="item" class="scroll-view-item">
    {{ item }}
  </view>
</scroll-view>
```

```scss
#scroll-view{
  width: 100%;
  box-sizing: border-box; // 防止width为100%时总体宽度超出父盒子
  background: burlywood;
  padding: 16rpx 30rpx; // 左右padding为30rpx，滚动到两边时用padding隔开边缘
  display: flex;
  gap: 30rpx; // item的间隔，与左右padding相等
  .scroll-view-item{
    padding: 0 24rpx;
    font-size: 24rpx;
    line-height: 100rpx;
    border: 1rpx solid #E5E5E5;
    // 宽度明确指定时需要加上flex-shrink: 0，防止宽度被压缩为fit-content
    // width: 200rpx; 
    // flex-shrink: 0;
  }
}
```

**兼容性问题**：
- ❌ **微信小程序**：需要添加 `enable-flex` 属性才能支持 flex 布局，且存在以下问题：
  - `gap` 属性不生效
  - item 高度异常
  - 整体布局行为不稳定
- ✅ **支付宝小程序**：运行正常

## 总结
建议使用**推荐方案（写法二）**，通过内层包裹容器的方式实现横向滚动，避免了直接在 scroll-view 上使用 flex 布局导致的兼容性问题。
