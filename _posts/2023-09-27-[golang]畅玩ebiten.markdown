---
layout: post
title:  "[golang]学习:畅玩ebiten"
date:   2023-09-27
categories: golang	ebiten
---

ebiten基本函数学习

## 介绍

 *Ebiten* 是一个用go语言编写的开源的跨平台的2D游戏引擎。

### Part1 绘制基本图形

- **显示游戏窗口**

创建一个空白工程，然后就像导入其他go package一样导入ebiten

```
    package main
    import (
    "log"
    "github.com/hajimehoshi/ebiten/v2"
    )
```

现在我们需要创建一个Game结构体并为其构造一个创建函数，ebiten引擎运行时要求传入一个Game对象，此对象必须实现*ebiten.Game*这个接口：

```
// Game defines necessary functions for a game.
type Game interface {
  Update() error
  Draw(screen *Image)
  Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int)
}
```

其中有三个成员*Update()* ,*Draw()*,*Layout()*

1. Update：每个tick都会被调用。tick是引擎更新的一个时间单位，默认为1/60s。tick的倒数我们一般称为帧，即游戏的更新频率。默认ebiten游戏是60帧。
2. Draw：每帧（frame）调用。帧是渲染使用的一个时间单位，依赖显示器的刷新率。如果显示器的刷新率为60Hz，`Draw`将会每秒被调用60次。`Draw`接受一个类型为`*ebiten.Image`的screen对象。ebiten引擎每帧会渲染这个screen。在上面的例子中，我们调用`ebitenutil.DebugPrint`函数在screen上渲染一条调试信息。由于调用`Draw`方法前，screen会被重置，故`DebugPrint`每次都需要调用。
3. Layout：该方法接收游戏窗口的尺寸作为参数，返回游戏的逻辑屏幕大小。我们实际上计算坐标是对应这个逻辑屏幕的，`Draw`将逻辑屏幕渲染到实际窗口上。这个时候可能会出现伸缩。

```
const (
	screenWidth  = 300
	screenHeight = 300
)

type Game struct{}

func (g *Game) Update() error {
  return nil
}
//每1/60秒会被调用一次，屏幕绘制（画图背景文字什么的）
func (g *Game) Draw(screen *ebiten.Image) {
  ebitenutil.DebugPrint(screen, "Hello, World")
}
//每1/60秒会被调用一次，显示的是游戏逻辑范围
func (g *Game) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
  return screenWidth, screenHeight
}

func main() {
  //游戏显示背景（全部的可视范围）
  ebiten.SetWindowSize(screenWidth*2, screenHeight*2)
  //title标题设置
  ebiten.SetWindowTitle("这是一个标题")
  if err := ebiten.RunGame(&Game{}); err != nil {
		panic(err)
	}
}
```

![image](/assets/part1.png)

我们创造了一个`Game`结构体实施的是`ebiten.Game`的接口，其中`Game`必须有2个函数`Layout`以及`Update`，而最好添加`Draw`函数，因为他是控制图像显示的函数，相当于电脑显示器的作用。

`screenWidth`和`screenHeight`为逻辑屏幕的大小，而其中`ebiten.SetWindowSize`设置的是全部可见范围的屏幕大小。

在上面的例子中游戏窗口大小为(300, 300)，`Layout`返回的逻辑大小为(600, 600)，所以显示会放大1倍。

- **绘制图形**

我们可以写一些简单的代码来绘制一些简单的图形：

`screen.Set()`为像素填充函数设置好XY的像素位置和颜色就可以填充了

紫色的正方形：

```
func (g *Game) Draw(screen *ebiten.Image) {
	purpleCol := color.RGBA{255, 0, 255, 255}
	for x := 100; x < 200; x++ {
		for y := 100; y < 200; y++ {
			screen.Set(x, y, purpleCol)
		}
	}
}
```

我们可以用同样的方法来绘制圆形：

```
func (g *Game) drawCircle(screen *ebiten.Image, x, y, radius int, clr color.Color) {
	radius64 := float64(radius)
	minAngle := math.Acos(1 - 1/radius64)
	for angle := float64(0); angle <= 360; angle += minAngle {
		xDelta := radius64 * math.Cos(angle)
		yDelta := radius64 * math.Sin(angle)
		x1 := int(math.Round(float64(x) + xDelta))
		y1 := int(math.Round(float64(y) + yDelta))
		screen.Set(x1, y1, clr)
	}
}
func (g *Game) Draw(screen *ebiten.Image) {
	purpleClr := color.RGBA{255, 0, 255, 255}
	g.drawCircle(screen, 150, 150, 50, purpleClr)
}
```

![image](\assets\circle.png)
