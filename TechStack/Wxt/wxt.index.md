wxt可以使用ts。
wxt采用的是约定式路由，自动生成manefest.json文件，不需要显式去修改。所有生成的文件放在.output文件夹下。
## 搭建流程

初始化项目，并选择vue模板
```bash
pnpm dlx wxt@latest init <project-name>
```
进入目录后安装依赖
```bash
pnpm i
```
## feat: newtab
浏览器安装插件后，插件可以修改浏览器新建标签页默认打开的页面。
在entrypoints文件夹下新建newtab文件夹，并定义index.html, main.ts, App.vue等必要的文件即可。
## 引入scss
wxt提供的vue模板没有把scss集成进来，导致在vue文件的style块不能使用scss语法。直接安装sass即可解决。
```bash
pnpm i -D sass
```
## 引入unocss
安装unocss
```bash
pnpm add -D unocss
```
新增uno.config.ts
```ts
import { defineConfig } from 'unocss'
export default defineConfig({
  // UnoCSS options
})
```
修改wxt.config.ts
```ts
import { defineConfig } from 'wxt';
import UnoCSS from 'unocss/vite'
// See https://wxt.dev/api/config.html

export default defineConfig({
  modules: ['@wxt-dev/module-vue'],
  vite: () => ({
    plugins: [UnoCSS()],
  }),
});
```
在各个页面中引入unocss
```ts
// main.ts

import 'uno.css'
 
import { createApp } from 'vue';
import './style.css';
import App from './App.vue';
createApp(App).mount('#app');
```
unocss官方预设presetUno是TailwindCSS, WindiCSS的超集，所以TailwindCss 的类在这里都能正常使用，而且可以使用h-10px等WindiCSS的语法。