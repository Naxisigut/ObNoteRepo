- 构建一个ExtFieldsHandler类实例，一个实例对应一个表格。
- 通过这个实例，对数据的扩展字段进行展开、回写等操作，完成数据显示、保存等功能
## 引入
```js
import { newExtFieldsHandler } from '@/utils/extFields.js';

data(){
	extFieldsHandler: {}, // 扩展字段
	extBusinessType: 2, // 扩展字段业务type 销售发货单：2
}
```

## 构建
- extFieldsHandler类实例只有一个属性extFields，即当前表格的扩展字段， 在构建时需要传入表格对应的extFields
- 表格对应的extFields通过给GetInitialExtFieldByTableCode接口传参tableCode获取
- 构建时机是在进入页面后的created生命周期内， 越早越好

```js
created () {
  // ...
  this.getExtFieldsHandler() // 初始化扩展字段
  // ...
},

methods: {
  getExtFieldsHandler(){
    newExtFieldsHandler(this.tableCode, 3).then((res) => {
      this.extFieldsHandler = res
    })
  },
}
```

## Template
- gyl-table需要传入扩展字段插槽来正常显示数据
- 扩展字段统一使用input输入框来编辑
- TODO：非编辑状态

详情：
```html
<gyl-table>
	<!-- ... -->
	<!-- 扩展字段 -->
	<template v-for="ext in extFieldsHandler.extFields" :slot="ext.Code" slot-scope="scope">
    <div :key="ext.Code">
      <el-input v-if="tableInpuData[ext.Code] && tableInpuData[ext.Code].IsEdit" v-model="scope.data.row[ext.Code]" class="tableCusInput" v-direction:a="{x: scope.index, y: scope.data.$index }">
      </el-input>
    <span v-else>{{ scope.data.row[ext.Code] }}</span>
    </div>
  </template>
	<!-- ... -->
</gyl-table>
```

查询：
```HTML
<gyl-table>
	<!-- ... -->
	<!-- 扩展字段 -->
	<template v-for="ext in extFieldsHandler.extFields" :slot="ext.Code" slot-scope="scope">
		<span :key="ext.Code" >{{ scope.data.row[ext.Code] }}</span>
	</template>
	<!-- ... -->
</gyl-table>
```

## 显示数据
- 获取到表格数据后，需要调用extendList将扩展字段展开
- extendList方法会返回一个新的数组，需要通过赋值this.tableData来获取响应式
```js
queryPOUnreceivingTable () {
	// ...
	if (res.data.success) {
    this.tableData = res.data.entity.PageList.data;
	  if (this.searFormData.OrderId > 0) {
	    // ...
      this.tableData = this.extFieldsHandler.extendList(this.tableData)
      this.queryTotalSum();
    }
  }
  // ...
}
```
## 新增明细
- 新增明细一般在batchJoin方法里。
- 新增明细进入表格数据，需要使新增的明细数据与原有的数据在扩展字段的结构上保持一致
- 有两种情况：
	1. 手动赋值添加的数据，没有初始的ExtFields字段，可以在原有赋值结果的基础上调用initRow方法并重新赋值
	2. 从其它表格引入的数据，有自身的ExtFields字段，则需要调用transferList方法并带入数据中原有的赋值
	3. 若不需要带入源数据的值，则不用给transferList的第二个参数

```js
// 情况一
if (this.searFormData.OrderId > 0) {
	this.packageInfoData = res.data.entity.PackageSpecs;
	let objData = {};
	for (let obj in this.tableData[0]) {
		objData[obj] = '';
	}
	objData.SortNo = this.tableData[this.tableData.length - 1].SortNo + 1;
	objData = this.extFieldsHandler.initRow(objData) 
	this.tableData.push(objData)
}
```

```js
// 情况二
batchJoin (e, p, idx) {
  let msg = false
	e = this.extFieldsHandler.transferList(e, {
    fromBusinessType: 1,
    toBusinessType: this.extBusinessType  
  }) // *转换扩展字段*
	e.forEach((item, index) => {
    item.AuxiliaryUnit = item.AuxiliaryUnit == 0 ? '' : item.AuxiliaryUnit.toString()
    this.tableData.push(item);
  })
  // ...
}
```
## 保存
- 保存前需要将展开的扩展字段值回写到ExtendFields里，通过调用collectList完成
```js
async saveSaleOrder () {
	var sort = 1;
    let searFormAreaCode = ',' + this.searFormData.AreaCode.toString() + ',';
    let params = {
	  // ...
      TaxesDiscount: this.searFormData.TaxesDiscount,
      Details: [],
      CreateDate: this.$moment(this.searFormData.CreateDate).format('YYYY-MM-DD'),
    }
    this.extFieldsHandler.collectList(this.tableData)
    for (let i = 0; i < this.tableData.length; i++) {
      if (this.tableData[i].BrandProductId > 0) {
        let detailInfo = {
          SortNo: sort++,
          // ...
          ProductionRemark: this.tableData[i].ProductionRemark,
          ExtFields: this.tableData[i].ExtFields // 扩展字段
        }
        params.Details.push(detailInfo);
      }
    }
}
```