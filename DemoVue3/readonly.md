readonly方法与reactive一样，用于代理一个对象。不同在于无法通过这个代理去修改源对象。
## 单测
```ts
it('happy path', () => {
    const target = { foo: 1 }
    const proxy = readonly(target)
    expect(proxy).not.toBe(target)
    expect(proxy.foo).toBe(1)

    // cannot be set
    proxy.foo = 2
    expect(proxy.foo).toBe(1)
})
```
## 实现
因为readonly的源对象的值不会被改变，所有没有必要track，也没有必要trigger。
```ts
export function readonly(target){
  const _proxy = new Proxy(target, {
    get(target, key){
      const res = Reflect.get(target, key)
      return res
    },
    set(target, key, value){
      return true
    }
  })
  return _proxy
}
```
## 重构
