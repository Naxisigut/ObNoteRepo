## next/navigation
#### useRouter
1. 编程式操作路由
2. `use client` 
```tsx
'use client'
import { useRouter } from 'next/navigation'
 
export default function Page() {
  const router = useRouter()
  return (
    <button 
      type="button" 
      onClick={
        () => router.push('/dashboard')
      }
    >
      Dashboard
    </button>
  )
}
```

#### usePathname
1. 获取当前Url路径
2. `use client` 
```tsx
'use client'
 
import { usePathname } from 'next/navigation'
 
export default function ExampleClientComponent() {
  const pathname = usePathname()
  return <p>Current pathname: {pathname}</p>
}
```
#### useParams
返回一个对象，包含当前路径中所有的动态参数。
#### useSearchParams
返回一个只读的URLSearchParams对象，可以获取当前路径上所有的query参数。