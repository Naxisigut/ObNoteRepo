## 参考资料
https://cn.vitejs.dev/guide/build.html#library-mode

## 初始化
```bash
pnpm create vite
```
```bash
选择vanilla
```
## vite.config.js
在库模式中进行配置
https://cn.vitejs.dev/guide/build.html#library-mode
```js
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'


export default defineConfig({
  server: {
    port: 5100,
  },
  build: {
    lib: { // 开启库模式
      entry: resolve(__dirname, 'src/index.ts'), // Could also be a dictionary or array of multiple entry points
      name: 'lutils',
      // the proper extensions will be added
      fileName: 'lutils'
    }
    // rollupOptions: {
    //   // 确保外部化处理那些你不想打包进库的依赖
    //   external: ['vue'],
    //   output: {
    //     // 在 UMD 构建模式下为这些外部化的依赖提供一个全局变量
    //     globals: {
    //       vue: 'Vue',
    //     },
    //   },
    // },
  }
})


```

## package.json
- 添加库信息
- 安装vitest，vitest ui
- 配置脚本

```json
{
  "name": "lutils",
  "private": false,
  "version": "0.0.0",
  "type": "module",
  "files": [
    "dist"
  ],
  "main": "./dist/lutils.umd.cjs", // 构建产物
  "module": "./dist/lutils.js", // 构建产物
  "exports": {
    ".": {
      "import": "./dist/lutils.js",
      "require": "./dist/lutils.umd.cjs"
    }
  },
  "scripts": {
    "dev": "vitest --ui", // 将vitest ui的界面作为开发服务器
    "test": "vitest", // 命令行vitest
    "build": "tsc && vite build", // 打包库
    "preview": "vite" // 在vite的开发服务器的index.html中试用库
  },
  "devDependencies": {
    "@vitest/ui": "^2.1.1",
    "typescript": "^5.5.3",
    "vite": "^5.4.1",
    "vitest": "^2.1.1"
  },
  "dependencies": {
    "@types/node": "^22.7.4"
  }
}

```
## trick
#### vite.config.ts文件ts报错
报错信息为 `找不到名称“__dirname”`, `找不到模块 “path“ 或其相对应的类型声明`
只要安装node的类型声明即可。
```
pnpm i @types/node -d
```