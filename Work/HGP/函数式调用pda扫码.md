## 说明
原本的pda调用：
1. 引入xw-scan组件并在模板中使用
2. 在生命周期中使用`uni.$on`, `uni.$off`对xwscan事件进行监听或移除监听，并做相应处理
现在改造为函数式的调用，更为灵活。
注意pda只需要在安卓app实现，其它环境不报错即可。

## 实现
### 函数式调用pda
实现pda的函数式调用，并包装为vue插件的形式。
```js
const PDA = {
  init: () => {},
  setHandler: () => {},
  test: () => {}
}

function toast(msg){
  uni.showToast({
    title: msg,
    icon: 'none',
    duration: 1500
  })
}

//  #ifdef APP
let pdaHandler = null
function setHandler(cb){ // 设置handler
  pdaHandler = cb;
}

const main = plus.android.runtimeMainActivity()
let receiver = null
function stop(){
  if(receiver) main.unregisterReceiver(receiver);
  receiver = null
}
function init(){
  // 初始化&恢复初始状态，使用最新的pda设置重新注册receiver
  // 即使接收到广播，也不会触发handler
  if(receiver) stop();
  pdaHandler = null

  const pdaSettingInfo_Json = uni.getStorageSync("pdaSettingInfo");
  if(!pdaSettingInfo_Json){
    // toast('获取PDA设置失败');
    console.log('获取PDA设置失败');
    return 
  } 
  const pdaSettingInfo = JSON.parse(pdaSettingInfo_Json);
  if(!pdaSettingInfo.BroadcastAction || !pdaSettingInfo.BroadcastTag){
    // toast('PDA设置不全');
    console.log('PDA设置不全');
    return 
  }

  let IntentFilter = plus.android.importClass('android.content.IntentFilter');
  const filter = new IntentFilter();
  //下面的addAction内改为自己的广播动作
  filter.addAction(pdaSettingInfo.BroadcastAction);
  let locker = false
  receiver = plus.android.implements('io.dcloud.feature.internal.reflect.BroadcastReceiver', {
    onReceive: (context, intent)=> {
      if(locker) return;
      locker = true;
      plus.android.importClass(intent);
      //下面的getStringExtra内改为自己的广播标签--有误
      let code;
      if(intent.getStringExtra(pdaSettingInfo.BroadcastTag) != null) {
        code = intent.getStringExtra(pdaSettingInfo.BroadcastTag); 
      }else {
        code = String.fromCharCode.apply(null, intent.getByteArrayExtra(pdaSettingInfo.BroadcastTag))
      }
      typeof pdaHandler === 'function' && pdaHandler({code});
      setTimeout(() => {
        locker = false;
      }, 150);
    }
  });

  main.registerReceiver(receiver, filter);
}
// 测试用，调用后会触发onReceive方法
function test(msg){
  const Intent = plus.android.importClass('android.content.Intent');
  const pdaSettingInfo = JSON.parse(uni.getStorageSync("pdaSettingInfo"));  
  const intent = new Intent();
  intent.setAction(pdaSettingInfo.BroadcastAction);
  intent.putExtra(pdaSettingInfo.BroadcastTag, msg);
  main.sendBroadcast(intent);
}


PDA.init = init;
PDA.setHandler = setHandler;
PDA.test = test
// #endif
// #ifndef APP

// #endif

export default {
  install: (Vue) => {
    Vue.prototype.$pda = PDA;
  }
}
```
## 使用
### 引入插件
```js
// main.js
import PDA from './tool/pda.js'
Vue.use(PDA);
```

### 初始化
由于每个页面使用pda要完成的业务不同，所以每次进入或退出页面时最好重置/初始化pda对象。
这里选择在退出页面、进入workbench页面时进行初始化。
```js
// workbench.vue
onShow() {
  this.$pda.init();
},
```
### 调用
每次进入页面在生命周期绑定回调。
回调可以加锁做节流。
```js
onShow() {
  let isLock = false
  this.$pda.setHandler((res)=> {
    if(isLock)return
    isLock = true
    if(!res.code)return this.$msg('扫描失败！请前往【我的】-【PDA配置】检查是否启用或配置PDA默认配置！', 4000)
    this.inputValue = res.code
    this.iptScan()
    setTimeout(() => isLock = false, 100)
  });
},

```
