## Suspense
在其包裹的子组件加载完成前展示临时填充的视图。
#### props
- `children`: 真正需要展示的视图
- `fallback`: 后备视图
#### 注意
1. `fallback`在子组件处于加载（挂起）状态时展示，比如首次进入，或者子组件状态改变导致被再次挂起
2. 若子组件再次被挂起是由 `startTransition` / `useDeferredValue` 引起，那么`fallback`不会被展示
3. 只有启用了`Suspense`的数据源才会使该组件生效，如支持`Suspense`的框架Nextjs, 使用`lazy`懒加载的组件代码，使用`use`读取Promise的值。
#### 用法
```tsx
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```
