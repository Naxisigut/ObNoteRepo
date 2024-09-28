## 参考资料
https://cn.vitejs.dev/guide/build.html#library-mode

## 过程
```
pnpm create vite
vanilla
```
## trick
#### vite.config.ts文件ts报错
报错信息为 `找不到名称“__dirname”`, `找不到模块 “path“ 或其相对应的类型声明`
只要安装node的类型声明即可。
```
pnpm i @types/node -d
```