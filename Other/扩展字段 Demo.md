- 构建一个extFieldsHandler类实例，一个实例对应一个表格。
- 通过这个实例，对数据的扩展字段进行展开、回写等操作，完成数据显示、保存等功能
## 引入
- 引入：
```js
import { ExtFieldsHandler } from '@/utils/extFields.js';

data(){
	extFieldsHandler: {} // 扩展字段
}
```

## 构建
- extFieldsHandler类实例只有一个属性extFields，即当前表格的扩展字段
- 构建时传入表格表头，会自动抓取其中的扩展字段
- 所以构建的时机是在获取表格表头之后，一般就是this.allDataObj.TableFields，或者this.allDataObj.TableFields

详情：
```js
//获取表格是否显示/编辑
    querySalesOrderDetailProductFieldProperty (e) {
      getSalesOrderDetailProductFieldProperty(this.orderId).then(res => {
        // ...
        this.allDataObj.TableFields = e;
        this.extFieldsHandler = new ExtFieldsHandler(e) // 初始化扩展字段
		// ...
    },
```

查询：
```js
//获取form查询表单和table表头数据
queryCustamTable() {
	// ...
	this.allDataObj = res.data.entity;
	this.$set(this.allDataObj, "TableFields", res.data.entity.TableFields );
	this.$set( this.allDataObj, "SearchFormFields", res.data.entity.SearchFormFields );
	this.extFieldsHandler = new ExtFieldsHandler(this.allDataObj.TableFields) // 初始化扩展字段
	this.queryPOUnreceivingTable();
	// ...
},
```

## Template
- gyl-table需要传入扩展字段插槽来正常显示数据
- 扩展字段统一使用input输入框来编辑
- TODO：非编辑状态

详情：
```html
<gyl-table :tableData="tableData" :tableCol="allDataObj.TableFields" :sortStatus="false" :checkedStatus="true" @updateSelectList="updateSelectList" :totalSum="totalSum" :totalStatus="true" :isSortable="false" class="orderInfo_table" ref="orderInfoTable">
	<!-- ... -->
	<!-- 扩展字段 -->
	<template v-for="ext in extFieldsHandler.extFields" :slot="ext.FieldCode" slot-scope="scope">
		<el-input v-if="tableInpuData[ext.FieldCode] && tableInpuData[ext.FieldCode].IsEdit" v-model="scope.data.row[ext.FieldCode]" class="tableCusInput" v-direction:a="{x: scope.index, y: scope.data.$index }">
		</el-input>
		<span v-else>{{ scope.data.row.AuxiliaryAmount }}</span>
	</template>
	<!-- ... -->
</gyl-table>
```

查询：
```HTML
<gyl-table :tableData="tableData" :tableCol="allDataObj.TableFields" :sortStatus="false" :checkedStatus="true" @updateSelectList="updateSelectList" :totalSum="totalSum" :totalStatus="true" :isSortable="false" class="orderInfo_table" ref="orderInfoTable">
	<!-- ... -->
	<!-- 扩展字段 -->
	<template v-for="ext in extFieldsHandler.extFields" :slot="ext.FieldCode" slot-scope="scope">
		<span :key="ext.FieldCode" >{{ scope.data.row[ext.FieldCode] }}</span>
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
- 新增明细进入表格数据，需要使新增的明细数据与原有的数据在扩展字段的结构上保持一致
- 有两种情况：
	1. 手动赋值添加的数据，没有初始的ExtFields字段，可以在原有赋值结果的基础上调用initRow方法并重新赋值
	2. 从其它表格引入的数据，有自身的ExtFields字段，则可以调用transferList方法并重新赋值

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
	e = this.extFieldsHandler.transferList(e) // **
	e.forEach(item => {
		this.tableData.forEach(cIm => {
			if (item.BrandProductId == cIm.BrandProductId) {
				msg = true
	            return false
		     }
	     })
	})
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