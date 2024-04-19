## 描述
1. 使用方法和类命名规范尽量与TailwindCss保持一致
2. 一次引入，所有页面可用。

## 使用方法
1. 在App.vue不添加scoped的style内引入scss文件
```html
<style lang="scss">
@import "./style/atom-class.scss";
</style>
```
2. 在任意页面使用。
3. 使用过程中可在TailwindCss官网直接搜索


## 类目录
由于原子类太多，目前是用到哪些类就添加哪些类。
更新至20240117。
1. margin & padding
2. weight & height
3. line-height
4. font-size & font-weight & color & text-align
5. background-color
6. border-radius
7. border-width & border-style & bordor-color
8. flex相关
9. grid相关
10. display
11. overflow
12. object-fit
13. aspect-ratio
14. poly-class聚合类

## Best Practices

