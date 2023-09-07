## 例子：销售订单详情
1. 由 `salesOrderDetailField` 控制（业务）
2. `querySalesOrderDetailFieldProperty` 方法获得 `salesOrderDetailField`
	1. 接口：`getSalesOrderDetailFieldProperty`
	2. 字段：`IsEdit` 字段，取反为输入框的 `disabled`

改为：
1. 由 `salesOrderDetailField`（业务）和 `this.allDataObj.DetailFormFileds`（个性化）共同控制，同一个字段两者的`IsEdit`同时为`true`时才不禁用，其它情况均禁用。
2. `querySalesOrderDetailFieldProperty` -> `salesOrderDetailField`
	1. 接口：`GetSalesOrderDetailFieldProperty`
	2. 字段：`IsEdit`
3.  `queryCustamForm` -> `this.allDataObj.DetailFormFileds`
	1. 接口：`GetUserCustomizDetailFormFiled`
	2. 字段：`IsEdit`
## 方案
#### mixin & 引入mixin
```ts
// 主档表单业务字段属性统一命名为 formFieldProperty
// 获取formFieldProperty的方法 统一命名为 getPODetailFieldProperty

export default{
  computed:{
    // 主档表单字段禁用控制
    fileFieldDisableHandler(){
      const handler = {}
      const customSource = this.fileFieldCustomMap // 个性化字段属性
      const fileStatusSource = this.formFieldProperty  // 业务字段属性
      const fields = Object.keys(this.fileFieldCustomMap)
      for (let i = 0, len = fields.length; i < len; i++) {
        const field = fields[i]
        // 个性化和业务的IsEdit同时为true时，disable为false
        handler[field] = !customSource[field].IsEdit || !fileStatusSource[field].IsEdit 
      }
      return handler
    },
    // 主档表单字段个性化属性对象
    fileFieldCustomMap(){
      const source = this.allDataObj.DetailFormFileds
      if(!source)return {}
      const map = {}
      for (let i = 0, len = source.length; i < len; i++) {
        const item = source[i];
        map[item.ColumnCode] = item
      }
      return map
    }
  },
}
```
```ts
import disableHandlerMixin from "@/mixins/disableHandlerMixin";
export default {
  mixins: [disableHandlerMixin],
}
```
#### 使用fileFieldDisableHandler
```html
<template slot="StockTakingNo" slot-scope="scope">
  <el-input 
    v-model="searFormData.StockTakingNo" 
    placeholder="请输入入库单号" 
    :disabled="fileFieldDisableHandler['StockTakingNo']"
    clearable 
  ></el-input>
</template>
```
#### 获取主档业务字段属性
方法统一命名为 getPODetailFieldProperty
主档表单业务字段属性统一命名为 formFieldProperty
```js
getPODetailFieldProperty () { //获取表单默认编辑状态
  GetCycleTakeDetailFieldPropertyByFastener(this.id).then(res => {
    if (res.data.success) {
      this.formFieldProperty = res.data.entity
    }
  })
},
```
