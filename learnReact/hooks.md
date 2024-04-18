#### useTransition
1. 指定某些状态为低优先级，延后更新
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

#### useRef