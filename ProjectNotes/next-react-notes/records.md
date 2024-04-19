#### 异步服务器组件ts报错
代码如下：
```tsx
export default async function SideBar() {
  return (
    <>
      <section className="col sidebar">
        <Link href={'/'} className="link--unstyled">
          <section className="sidebar-header">
            <img className="logo" width="22px" height="20px" src="/favicon.ico" alt="" role="presentation"/>
            <strong>Next React Notes</strong>
          </section>
        </Link>
        
        <section className="sidebar-menu" role="menubar">
          {/* sidesearch field */}
        </section>
        <nav>
          {/* sidebar note list */}
        </nav>
      </section>
    </>
  )
}
```
报错如下：
![[Pasted image 20240415203447.png]]
官方要求将ts和@types/react升级到最新版本即可，实测没有效果。