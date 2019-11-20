### border

```css
div{
	width:0;
	height:0;
	border:20px solid transparent;//transparent设置背景透明
	bordertop:20px solid red;
}
```

### Icon Font

把字体当作图标

```
@font-face{
	font-family:Triangel;
	src:url('./triangel') format("woff")
}
.triangel:before{
	content:"\t666"
}
```



