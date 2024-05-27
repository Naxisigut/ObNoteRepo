### useTransition
指定某些状态为低优先级，延后更新。
可用于输入框输入时防止阻塞。
```tsx
import { useTransition, useState } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [inputVal, setInputVal] = useState('')
  const [filterItems, setFilterItems] = useState([])
  const source = [...] // fake data
  const inputHandler = (e) => {
    setInputVal(e.target.value)

    // filterItems的更新会被延后
    startTransition(()=>{
      setFilterItems(source.filter(i => i.includes(inputVal)))
    })
  }
  return (
    <div>
      <input onInput={ inputHandler }  />
      <div> { isPending ? 'loading' : 'loading over'}</div>
      {
        filterItems.map((item, index) => {
          return <p key={index}>{ item }</p>
        })
      }
    </div>
  )
}
```

### useRef
用于声明一个ref对象。常用于获取dom，或是用于创建一个在多次渲染时需要保持当时的值的变量。
参考： https://react.docschina.org/reference/react/useRef#useref
用法： `const ref = useRef(initialValue)`
- `initialValue`: 初始值。只在首次渲染时有用
- 返回一个对象，只有一个key: `current`
- `ref.current`在重新渲染时被储存，不会被改变（普通对象每次渲染都会重新被赋值）
- ref对象在同个组件的不同实例间不会被共享

注意：
1. `ref.current`可以手动赋值。但若持有一个用于渲染的对象，则不应当手动赋值。
2. `ref.current`手动改变时，不会重新渲染。因为ref是一个普通对象，React不会关注其变化。所以一般`useRef`不会用来包裹需要在界面上显示的变量。
3. 除了初始化，渲染期间不要读写`ref.current`，后果难以预测。一般在事件响应/effect中读写`ref.current`

#### Demo1: 计时器
```tsx
// 计时器
// timer在整个组件的生命周期中都需要保持状态，所以使用useRef
// 使用useState当然是可以的，只是会引起不必要的渲染
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        开始
      </button>
      <button onClick={handleStop}>
        停止
      </button>
    </>
  );
}
```
#### Demo2: 操作dom
```tsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

// 将ref对象传给一个元素的ref prop
// React会自动将dom元素赋给ref.current
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```
#### Demo3: 操作自定义组件的dom
需要配合`forwardRef`
```tsx
import { forwardRef } from 'react';
// 1. 使用forwardRef将需要获取dom的自定义组件包裹起来
const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});
export default MyInput;
```
```tsx
import { useRef } from 'react';

export default function Father() {
  const inputRef = useRef(null);
  // 2. 在父组件中像正常dom元素一样，将ref对象传给自定义组件的ref prop
  return (
    <>
      <MyInput ref={inputRef} />
    </>
  );
}
```

### useEffect

### useFormStatus
### useFormState