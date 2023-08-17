## 单测
- happy path: get set
```ts
it('ref happy path', ()=>{
    const proxy = ref(0)
    expect(proxy.value).toBe(0)
    proxy.value = 1
    expect(proxy.value).toBe(1)
  })
```

- 响应式：依赖
```ts
it('ref deps and effects', () => {
    const proxy = ref(0)
    let foo
    let count = 0
    effect(()=>{
      count++
      foo = proxy.value
    })
    
    // runner should be called when effect called
    expect(count).toBe(1)
    expect(foo).toBe(0)
  
    // effect should be triggered when target change
    proxy.value = 1
    expect(count).toBe(2)
    expect(foo).toBe(1)

    // same value should not trigger effect
    proxy.value = 1
    expect(count).toBe(2)
  })
```

- 响应式的嵌套
```ts
  it('ref value is nested', () => {
    const foo = ref({
      count: 1
    })
    let dummy
    effect(() => {
      dummy = foo.value.count
    })
    expect(dummy).toBe(1)
    foo.value.count = 2
    expect(dummy).toBe(2)
  })

})
```

## 实现
- RefImpl类
ref一般用于包裹简单类型，通过value来获取/设置；
通过实现一个RefImpl类来返回ref对象，value属性使用get/set方式来拦截；
在getter和setter内，分别对effect进行track和trigger；
ref.value所收集的依赖全部存放在ref.deps里面，方便查找和触发。

```ts
class RefImpl{
  private _value: any
  private _rawValue: any // 所代理的原始对象/值
  public deps: any
  constructor(value){
    // 对于对象，需要返回一个reactive代理对象
    this._value = convert(value)
    this._rawValue = value
    this.deps = new Set()
  }
  get value(){
    refTrack(this)
    return this._value
  }
  set value(newVal){
    // 若设置的值与当前值一样，则不触发副作用
    if(isEqual(this._rawValue, newVal))return
    this._value = convert(newVal)
    this._rawValue = newVal
    triggerEffect(this.deps)
    return
  }
}


export function ref(value){
  return new RefImpl(value)
}
```

- refTrack
在读取ref.value值时会进行track，但如果不是在effect内读取，此时track到的effect是undefined，所以需要加判断，只有在activeEffect存在时才进行track。这里调用了之前实现过的isTracking函数。
```ts
function refTrack(refImpl){
  if(isTracking())trackEffect(refImpl.deps)
}
```
- convert
若包裹的value是一个引用数据类型，比如对象，那么应该这个对象的属性也应该是响应式的。
所以这种情况下，在创建时应该返回一个reactive代理的对象。
同时在setter中也应有相同的逻辑
```ts
function convert(newVal){
  return isObject(newVal) ? reactive(newVal) : newVal
}

class RefImpl{
	constructor(value){
	    // 对于对象，需要返回一个reactive代理对象
	    this._value = convert(value)
	    this._rawValue = value
	    this.deps = new Set()
	 }
	 set value(newVal){
	    // 若设置的值与当前值一样，则不触发副作用
	    if(isEqual(this._rawValue, newVal))return
	    this._value = convert(newVal)
	    this._rawValue = newVal
	    triggerEffect(this.deps)
	    return
	  }
}
```