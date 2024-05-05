## 需求
1. 每个表格对应一个tableCode
2. 向组件传入tableCode, 表头和功能按钮tableCode配置好之后自动渲染
<!-- 每个表格的表头配置(`colsOption`)和按钮配置(`funcsOption`)都独立为一个对象，通过向`$getListCols`, `$getListFuncs`, `$getListStates` 传入`tableCode`获得 -->
3. 表格数据封装在组件内部
4. 需要分页

## Props
#### tableCode
表格Code
#### apis
需要用到的后端接口。apis.GetApi是查询表格数据的接口
#### searcher
查询条件对象。不需要传入分页。
#### usePager
是否需要分页。为false则不渲染分页，查询条件默认pageSize为9999条数据
#### pager & datas & loading
不使用内部数据时，需要传入分页参数对象，表格数据和loading状态参数，同时不能传入apis.GetApi
#### cols & funcs
表头配置对象和功能配置对象也可以使用外部传入，并且会被优先使用。

## Methods
#### search
请求数据。请求将在nextTick中发出。

## Event
#### command
接收所有自定义事件及其它参数s
