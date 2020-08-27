# 动效
## transform translate 
## animation @keyframe

# 选择器 


## 选择器列表 ，
## 基本选择器
元素选择器 p{}
类选择器    .class{}
ID选择器.  #nav{}
通配选择器*
属性选择器.
a[href^="#"] {
  background-color: gold;
}
## 关系选择器
相邻兄弟 +
通用兄弟 ～ 不用相邻
子选择器 直接后代  >
后代选择器。空格键


# z-index 失效   
(1)z-index属性只作用在被定位了的元素上。所以如果你在一个没被定位的元素上使用z-index的话，是不会有效果的.
(2)同一个父元素下的元素的层叠效果会受父元素的z-index影响,如果父元素的z-index值很小,那么子元素的z-index值很大也不起作用

失效情况|解决方法
:----|:-----
父元素position为relative|改为absolute
失效元素没有定位 没有position（不包括static）|添加position。relative absolute fix
失效标签含有浮动属性float	｜ 去除浮动
祖先标签z-index值比较小	｜ 提高祖父标签z-index 值
父元素的属性是position: sticky的时候,其子元素属性为position:absolute ｜ 	


# BFC
> 内部box 在垂直方向  一个接一个的放置 
> 属于同一个BFC的两个box  之间margin重叠
> bfc 不会与float box 重叠
> bfc 是页面一个隔离的容器 容器里面的子元素不会影响到外面
> 计算bfc高度时，浮动元素也参与计算 

+ 触发
> 绝对定位元素  absolute  fixed 
> 浮动元素  float 不为none 
> overflow 不为visible
> display 为 flex  inline-block table-cell table-caption

+ 应用场景 
> 高度崩塌 
> 阻止外边距重叠 
> 防止浮动元素遮住文档流中的元素  


# 清除浮动 
子元素浮动会造成父元素高度崩塌
> 父级定义height
> 只需要在浮动元素添加一个块级元素 添加空div clear:both
> 父级div定义伪类 :after zoom:1
> 父级div定义 overflow:hidden
