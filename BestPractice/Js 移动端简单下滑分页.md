# 移动端简单下滑分页实现

标签： #JavaScript #Vue3 #分页 #移动端 #Uniapp #下拉加载 #最佳实践

## 功能说明
实现移动端常见的下滑加载更多功能，包括：
- 🔄 **自动分页**：滑到底部自动加载下一页数据
- 🔃 **数据重置**：支持重新加载和刷新数据
- 📱 **移动端优化**：针对移动端交互优化
- 🛡️ **边界处理**：防止重复请求和无效加载

## 核心实现

### 模板结构
```html
<view v-if="list.length" class="list-wrapper" style="margin-top: 32rpx;">
  <view class="list-item" v-for="item in list" :key="item.id">
  </view>
  <view class="none_text">
    {{!isEnd?'加载中...':'暂时没有更多了...'}}
  </view>
</view>
<view v-else class="no-item-wrapper">
  <text class="">暂无记录</text>
</view>
```

### 状态管理
```js
const list = ref([])
const pageNumber = ref(1)
const pageSize = ref(20)
const isEnd = ref(false)
const getList = () => {
  const isFirst = pageNumber.value === 1
  return ApiName({
    pageNum: pageNumber.value,
    pageSize: pageSize.value 
  }).then(res => {
    if (res.success) {
      const oldList = isFirst ? [] : list.value
      list.value = oldList.concat(res.result) // 列表直接赋值，重置时不先置空
      isEnd.value = res.result.length < pageSize.value // 判断是否还有下一页数据
    } else {
      modal.toast(res.message)
    }
  })
}
```

### 数据重置
```js
// 重置
const resetList = () => {
  pageNumber.value = 1
  isEnd.value = false
  getList()
}
```

### 下滑加载
```js
// 下滑到底时请求下一页
onReachBottom(() => {
  if (!isEnd.value) {
    pageNumber.value++
    getList()
  }
})
```

### 页面生命周期
```js
// 进入页面初始化
const isLoaded = ref(false)
onLoad(async () => {
  await getRecordList()
  isLoaded.value = true
})
// 回到页面刷新数据
onShow(() => {
  pageScrollTo({ scrollTop: 0, duration: 200 });
  console.log('show begin');
  if(isLoaded.value){
    resetList()
  }
})
```

## 模板说明

### 核心功能
- **条件渲染**：`v-if="list.length"` 判断是否有数据
- **动态提示**：根据 `isEnd` 状态显示不同的加载提示
- **空状态处理**：无数据时显示友好提示

### 状态关联
- `list` - 渲染列表数据
- `isEnd` - 控制底部提示文案（"加载中..." / "暂时没有更多了..."）
- `item.id` - 作为列表项的 key 值

## 关键要点

### 1. 数据合并策略
- **首次加载**：直接替换列表数据
- **追加加载**：使用 `concat()` 合并新旧数据
- **避免闪烁**：不在追加时清空列表

### 2. 分页边界控制
- 通过 `isEnd` 标识判断是否还有更多数据
- 返回数据量小于 `pageSize` 时判定为最后一页
- 防止无效的后续请求

### 3. 页面状态管理
- 使用 `isLoaded` 标识区分首次进入和后续回显
- `onShow` 时重置滚动位置并刷新数据
- 确保数据的及时性和用户体验

## 使用场景
- 📋 **列表页面**：商品列表、文章列表等
- 📱 **移动应用**：小程序、H5页面
- 🔄 **数据流转**：需要分页加载的所有场景

## 注意事项
- 确保 API 接口支持分页参数（`pageNum`, `pageSize`）
- 根据实际 API 响应格式调整数据处理逻辑
- 建议添加加载状态提示，提升用户体验
