## 需求
box-shadow产生的阴影是整体的，无法对某个方向进行定制。在某些场景，需要盒子只显示单边阴影。
```css
.primary-btns-wrapper {
  /* layout */
  width: 100%;
  padding: 10upx 20upx;
  display: flex;
  box-sizing: border-box;
  background-color: #fff;
  box-shadow: 0 -10upx 20upx -10upx rgba(0, 0, 0, .1); // 阴影，与背景区分
  clip-path: polygon(100% 100%, 0% 100%, 0% -100%, 100% -100%); // 将左 右 下 的阴影裁掉
  button {
    background-color: #0066ff;
    color: #fff;
    line-height: 70upx; // 同时决定按钮高度
    font-size: 30upx;
    width: 0;
    flex: 1;
    & + button{
      margin-left: 40upx;
    }
  }
}
```