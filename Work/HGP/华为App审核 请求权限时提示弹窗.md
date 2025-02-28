## changeLog
v1.1 提示改为安卓原生弹窗
v1.2 新增基于`createRequestPermissionListener`的方案；函数式原生弹窗独立为一个文件

## 描述
华为应用市场的准入规则要求app在向用户请求权限时，需要以弹窗等形式提示用户请求该权限的用途，类似下图。
![[Pasted image 20240130170245.png]]

## 参考链接
https://en.uniapp.dcloud.io/api/system/create-request-permission-listener.html
https://blog.csdn.net/m0_73358221/article/details/133906138
https://blog.csdn.net/wz9608/article/details/135202285
https://blog.csdn.net/csdnxxxjjjqqq/article/details/127144435
![[华为App审核 请求权限时提示弹窗 方案1.excalidraw]]
![[华为App审核 请求权限时提示弹窗 方案2.excalidraw]]



## 实现函数式提示弹窗方法
```js
// permissionAlert.js
//安卓原生提示框
export const nativeObjView = ({title, content}) => {
  const systemInfo = uni.getSystemInfoSync();
  const statusBarHeight = systemInfo.statusBarHeight;
  const navigationBarHeight = systemInfo.platform === 'android' ? 48 :
    44; // Set the navigation bar height based on the platform
  const totalHeight = statusBarHeight + navigationBarHeight;
  let view = new plus.nativeObj.View('per-modal', {
    top: '0px',
    left: '0px',
    width: '100%',
    backgroundColor: '#444',
    //opacity: .5;
  })
  view.drawRect({
    color: '#fff',
    radius: '20px' // 弹窗圆角
  }, {
    top: totalHeight + 'px',
    left: '5%',
    width: '90%', // 弹窗宽度
    height: "90px",
  })
  view.drawText(title, {
    top: totalHeight + 15 + 'px',
    left: "8%",
    height: "30px"
  }, {
    align: "left",
    color: "#000",
    size: "20px",
    weight: "bold"
  }, {
    onClick: function (e) {
      console.log(e);
    }
  })
  view.drawText(content, {
    top: totalHeight + 35 + 'px',
    height: "60px",
    left: "8%",
    width: "84%"
  }, {
    whiteSpace: 'normal',
    size: "14px",
    align: "left",
    color: "#656563"
  })

  function show() {
    if (view) {
      view.show();
    } else {
      console.error("View is not initialized or has been destroyed.");
    }
  }

  function close() {
    if (view) {
      view.close();
      view = null; // 关闭后置为 null
    }
  }

  return {
    show,
    close
  }
}
```

## 实现方案1：Hbuilder>4.0
uniapp官方的解决方案`createRequestPermissionListener`需要
1. Hbuilder4.0+。
2. 只在安卓APP环境生效
3. 有一些bug存在，包括打包

### 引入函数式提示弹窗方法
```js
import { nativeObjView } from '@/common/permissionAlert.js';
```

### 在生命周期监听权限弹窗
```js
data() {
  return {
    permissionListener: null,
    permissionView: null
  }
},
onReady() {
  // #ifdef APP-PLUS
  this.watchPermission()
  // #endif
},
onUnload() {
  if (this.permissionListener) {
    this.permissionListener.stop()
  }
},
methods: {
  // 监听系统权限提示
  watchPermission() {
    this.permissionListener = uni.createRequestPermissionListener();
    this.permissionListener.onConfirm((e) => {
      let params = {}
      if(e.some(i => i.includes('CAMERA'))){
        params.title = '相机权限申请说明'
        params.content = '用于拍摄图片以提供客户服务'
      }else if(e.some(i => i.includes('WRITE_EXTERNAL_STORAGE'))){
        params.title = '存储等权限申请说明'
        params.content = '用于发送图片以提供客户服务'
      }
      this.permissionView = nativeObjView(params)
      this.permissionView.show()
    });
    this.permissionListener.onComplete((e) => {
      this.permissionView && this.permissionView.close()
    });
  },
}
```


## 实现方案2：扫码请求摄像头权限时提示用户弹窗-Hbuilder低于4.0
以下是4.0-的解决方案。
1. 由于无法实时监听请求权限弹窗，所以提示弹窗需要在请求权限之前出现。
2. H5环境无需请求权限。
3. 点击扫码图标 => 判断当前环境 => APP环境时判断是否拥有该权限 => 无权限时打开提示弹窗，再主动发起请求 => 授权，则扫码；无授权，则中止流程。

### 判断是否拥有某权限
```js
checkPermission(authorization){
  let compat = plus.android.importClass('androidx.core.content.ContextCompat')
  let context = plus.android.runtimeMainActivity()
  let result = compat.checkSelfPermission(context, authorization)
  // 0: 该权限已获授权
  return result === 0
},
```

### 主动请求某权限
简单地用Promise包装一下。
```js
  // authorizations 请求的权限数组
  // authorization 需要返回请求结果的权限名称
  requestPermission(authorizations, authorization) {
    return new Promise((resolve, reject) => {
      plus.android.requestPermissions(
        authorizations, 
        (resultObj) => {
          const granted = resultObj.granted
          const deniedPresent = resultObj.deniedPresent
          let res
          switch (true) {
            case granted.includes(authorization): 
              res = 1 // 允许
              break;
            case deniedPresent.includes(authorization):
              res = 0 // 本次拒绝（下次依然询问）
              break;
            default:
              res = -1 // 永久拒绝（不会显示询问弹窗）
              break;
          }
          resolve(res)
        }), 
        (e) => {
          reject(e)
        }
      })
  },
```

### 示例：工云链H5 扫码请求摄像头权限 preSacnMixin
1. 
```js
export default {
  data(){
    return {
      scanPermission: "android.permission.CAMERA",
      viewObjContent:{
        title: '摄像头权限说明',
        content: '摄像头权限将用于拍摄照片和视频。这样，您可以在应用程序中记录瞬间、分享内容或进行其他相关操作。'
      }
    }
  },
  methods:{
    /* 检查是否有权限 */
    checkPermission(authorization){
      let compat = plus.android.importClass('androidx.core.content.ContextCompat')
      let context = plus.android.runtimeMainActivity()
      let result = compat.checkSelfPermission(context, authorization)
      // 0: 该权限已获授权
      return result === 0
    },
    /* 发起权限请求并返回结果 */
    requestPermission(authorizations, authorization) {
      this.getHeight()
      return new Promise((resolve, reject) => {
        plus.android.requestPermissions(
          authorizations, 
          (resultObj) => {
            const granted = resultObj.granted
            const deniedPresent = resultObj.deniedPresent
            let res
            switch (true) {
              case granted.includes(authorization): 
                res = 1 // 允许
                break;
              case deniedPresent.includes(authorization):
                res = 0 // 本次拒绝（下次依然询问）
                break;
              default:
                res = -1 // 永久拒绝（不显示请求授权弹窗）
                break;
            }
            resolve(res)
          }), 
          (e) => {
            reject(e)
          }
        })
    },
    preScan() {
      return new Promise((resolve, reject) => {
        //#ifdef H5
        resolve(true)
        //#endif

        //#ifdef APP-PLUS
        const permitted = this.checkPermission(this.scanPermission)
        if(permitted){
          resolve(true)
        }else{
          const nativeView = this.nativeObjView(this.viewObjContent)
          const openTimer = setTimeout(() => {
            nativeView.show()
          }, 300)
          this.requestPermission(
            [ this.scanPermission ],
            this.scanPermission 
          ).then((res) => {
            if(res === -1){
              clearTimeout(openTimer)
              nativeView.close()
              this.$msg("请在系统设置中开启相机权限！")
            }else{
              clearTimeout(openTimer)
              nativeView.close()
              if(res === 1){
                resolve(true) 
              }else{
                resolve(false)
              }
            } 
          }).catch(() => {
            clearTimeout(openTimer)
            nativeView.close()
            resolve(false)
          })
        }
        //#endif
      })
    },
    //安卓原生提示框
    nativeObjView({title, content}) {
      const systemInfo = uni.getSystemInfoSync();
      const statusBarHeight = systemInfo.statusBarHeight;
      const navigationBarHeight = systemInfo.platform === 'android' ? 48 :
        44; // Set the navigation bar height based on the platform
      const totalHeight = statusBarHeight + navigationBarHeight;
      let view = new plus.nativeObj.View('per-modal', {
        top: '0px',
        left: '0px',
        width: '100%',
        backgroundColor: '#444',
        //opacity: .5;
      })
      view.drawRect({
        color: '#fff',
        radius: '5px'
      }, {
        top: totalHeight + 'px',
        left: '5%',
        width: '90%',
        height: "100px",
      })
      view.drawText(title, {
        top: totalHeight + 5 + 'px',
        left: "8%",
        height: "30px"
      }, {
        align: "left",
        color: "#000",
      }, {
        onClick: function (e) {
          console.log(e);
        }
      })
      view.drawText(content, {
        top: totalHeight + 35 + 'px',
        height: "60px",
        left: "8%",
        width: "84%"
      }, {
        whiteSpace: 'normal',
        size: "14px",
        align: "left",
        color: "#656563"
      })

      function show() {
        view = plus.nativeObj.View.getViewById('per-modal');
        view.show()
        view = null
      }

      function close() {
        view = plus.nativeObj.View.getViewById('per-modal');
        view.close();
        view = null
      }

      return {
        show,
        close
      }
    },
  }
}
```
