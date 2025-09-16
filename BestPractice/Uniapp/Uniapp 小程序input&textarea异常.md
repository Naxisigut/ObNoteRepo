标签： #Uniapp #input #textarea #小程序 #支付宝小程序 #最佳实践 #跨端兼容 #emoji

## 结论与要点
- **一键规避思路**：在出现输入相关异常的场景里，为 `input/textarea` 设置 `:enable-native="false"` 可显著改善问题。
- **副作用**：不确定是客观效果还是主观感受，添加该属性后程序运行更加流畅。
- **原理**：平台文档中缺少对该属性的描述，暂不清楚这个属性对于这些bug产生作用的机理。

## 异常清单

#### 弹窗中的输入框聚焦后高度异常
 - 平台：支付宝小程序（真机）
 - 正常：输入框应被键盘顶起并腾出可视区域
 - 异常：未顶起导致输入框被遮挡
 ![输入框顶起高度异常](attachments/input-textarea-bug-in-uniapp-1.png)

#### `textarea` 光标样式异常
 - 平台：支付宝小程序（真机）
 - 正常：聚焦状态下光标高度与行高一致且字符垂直居中；
 - 异常：字符相对光标明显不居中。
 ![输入框光标样式异常](attachments/input-textarea-bug-in-uniapp-2.png)

#### `textarea`&`input`中`emoji` 显示异常（支付宝）
 - 平台：支付宝小程序（真机）
 - 聚焦时 `emoji` 显示正常；
 - 失焦后 `emoji` 消失或不渲染。
 ![emoji显示异常](attachments/input-textarea-bug-in-uniapp-3.png)
 ![emoji显示异常](attachments/input-textarea-bug-in-uniapp-4.png)

## 解决方案
```vue
<template>
  <!-- #ifdef MP-ALIPAY -->
  <input :enable-native="false" placeholder="请输入" />
  <textarea :enable-native="false" auto-height placeholder="请输入描述" />
  <!-- #endif -->
</template>
```

## 注意事项
- 以真机为准进行验证，模拟器表现可能与真机存在差异。

## 参考
- [支付宝小程序input/textarea组件属性](https://opendocs.alipay.com/mini/component/input)