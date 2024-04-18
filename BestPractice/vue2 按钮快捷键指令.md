## 使用
#### 示例
```html
<el-button v-keycut.ctrl.s="loading" @click="checkSalePartNo()">添加明细</el-button>
```
`v-keycut.ctrl.s`: 指定匹配的快捷键, 支持ctrl/shift + 所有英文字母。英文字母最多一个，多余的不匹配。
`=loading`: data中的响应式变量。loading为true时不触发。
`@click=""`: 快捷键与click触发相同的事件。

注意：是否触发click事件目前由以下三个决定：
1. v-preventClick, 这个指令设置的3S节流同样作用在v-keycut上。
2. loading。为true时不触发。
3. 该指令内置1s防抖。