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
在各个页面中引入unocss。若已经定义了页面，但是又不引入，可能会在构建阶段警告`未引入uno.css`。
```ts
// main.ts
import 'uno.css'
import { createApp } from 'vue';
import './style.css';
import App from './App.vue';
createApp(App).mount('#app');


// background.ts
import 'uno.css';
export default defineBackground(() => {
  console.log('Hello background!', { id: browser.runtime.id });
});

// content.ts
import 'uno.css';
export default defineContentScript({
  matches: ['*://*.google.com/*'],
  main() {
    console.log('Hello content.');
  },
});
```
基础样式重置 http://unocss.cn/guide/style-reset.html
```bash
pnpm add @unocss/reset
```
```ts
// main.ts
import '@unocss/reset/tailwind-compat.css'
```
unocss官方预设presetUno是TailwindCSS, WindiCSS的超集，所以TailwindCss 的类在这里都能正常使用，而且可以使用h-10px等WindiCSS的语法。
## 引入shadcn-vue
引入shadcn-vue的过程比较复杂，主要是参考unocss-preset-shadcn的教程一步步来。参考资料：
- 官方教程
	https://www.shadcn-vue.com/docs/installation/vite
- unocss-preset-shadcn unocss社区预设
	https://github.com/hyoban/unocss-preset-shadcn#readme 
	[[unocss-preset-shadcn.README]]
- 参考项目
	https://github.com/mefengl/wxt-starter/tree/main
	https://github.com/hyoban-template/shadcn-vue-unocss-starter/tree/main

官方教程1 使用vite创建项目，省略 。
官方教程2 引入tailwindcss，由于使用的是unocss，省略。
官方教程3 在tsconfig.json里配置别名，由于wxt项目的目录结构不一样，省略。
官方教程4 由于是tailwindcss相关配置，省略
官方教程5 删除vite默认style文件，可省略。

安装`unocss-preset-shadcn`
```bash
pnpm i -D unocss-preset-animations unocss-preset-shadcn
```
配置unocss.config.ts，从[[unocss-preset-shadcn.README]]中复制。
```ts
import { defineConfig, presetUno } from 'unocss'
import presetAnimations from 'unocss-preset-animations';
import { presetShadcn} from 'unocss-preset-shadcn';

export default defineConfig({
  // UnoCSS options
  presets: [
    presetUno(),
    presetAnimations(),
    presetShadcn({
      // color: 'red',
      // With default setting for SolidUI, you need to set the darkSelector option.
      // darkSelector: '[data-kb-theme="dark"]',
    }),
  ],
  content: {
    pipeline: {
      include: [
        // the default
        /\.(vue|svelte|[jt]sx|mdx?|astro|elm|php|phtml|html)($|\?)/,
        // include js/ts files
        '(components|src)/**/*.{js,ts}',
      ],
    },
  },
})
```
初始化`shadcn-vue`：安装社区预设依赖。注意不要按照官方教程步骤6。
```bash
// !!! do not use pnpx shadcn-vue@latest init !!!

pnpm i lucide-vue-next radix-vue class-variance-authority clsx tailwind-merge
```
复制utils.ts到项目根目录lib文件夹下，这个文件是shadcn组件需要使用的。
```ts
// utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```
在项目根目录下添加components.json，这个文件在shadcn安装组件时需要使用，指定了安装信息。
```json
{
  "style": "default",
  "typescript": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "src/assets/index.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "framework": "vite",
  "aliases": {
    "components": "/components", // 组件安装位置
    "utils": "@/lib/utils" // utils文件相对位置
  }
}
```
