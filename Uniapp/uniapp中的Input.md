## 基本使用
``` html
<input type="text" v-model="value">

```
``` html
<!-- 错误写法 -->
<!-- 与PC平台的Vue不同，以下写法不会实时更新绑定值 -->
<input type="text" :value="value" @input="handleInput">

<script>
  // ...
  data(){
    return{
      value: ''
    }
  },
  methods:{
    handleInput(e){
      this.value = e.detail.value
    }
  }
</script>
```
## type
type为number时，输入小数点`.`会触发input事件，但input事件处理函数获得的参数`e.datail.value`和实时更新的绑定值均不包含小数点`.`（H5）