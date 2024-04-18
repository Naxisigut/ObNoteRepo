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