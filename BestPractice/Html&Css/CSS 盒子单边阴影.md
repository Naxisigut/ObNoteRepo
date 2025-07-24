#CSS 
## 需求
box-shadow产生的阴影是整体的，无法对某个方向进行定制。在某些场景，需要盒子只显示单边阴影，比如说放在底部的按钮组，需要通过阴影与背景区分，同时两侧又需要和背景一体。

## 实现
思路：
1. box-shadow 垂直向上偏移，同时灵活控制扩散半径
2. clip-path裁剪出可视区域
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