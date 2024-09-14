#CSS 
#### 要求
1. 盒子本身是个可滚动盒子
2. 浏览器最好是chrome内核的浏览器，否则可能有兼容性问题
#### 实现
```css
  .box{
    width: 200px;
    height: 200px;
    overflow-y: auto;
  }
  .box::-webkit-scrollbar {
    width: 6px; /* 滚动条宽度 */
  }
  .box::-webkit-scrollbar-thumb{
    border-radius:10px; /* 滚动条两端圆角 */
    background-color: rgba(200, 200, 200, .9); /* 滚动条颜色 */
  }
```