不要直接操作pages.json，其内容会自动生成。
- 使用约定式路由，pages文件夹下的所有vue文件会单独生成一个页面
- 全局的设置在pages.config.ts中完成
- 页面的设置在页面组件的route代码块中完成
 
不要直接操作manifest.json，其内容会自动生成。
- 需要配置的内容请在 `manifest.config.ts` 里面配置。
## unocss的使用
类名中的数值可以使用`px`, `rpx`等单位，比如`py-52rpx`, `text-24px`。
