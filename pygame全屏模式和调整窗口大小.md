# ``pygame``如何实现全屏模式和窗口大小可调

在``pygame``中可以设置通过``pygame.display.set_mode(size=(0, 0), flags=0, depth=0)``对窗口参数进行设置，这个函数会返回一个``Surface``对象。

在四个参数中，``size``表示窗口大小，``flags``参数可以控制窗口模式，``depth``参数表示颜色的位数

``flags``参数：

- ``pygame.FULLSCREEN`` 创建全屏的窗口
- `` pygame.DOUBLEBUF`` 使用HWSURFACE或OPENGL时建议加上这个标志
- ``pygame.HWSURFACE`` 使用硬件加速，只在FULLSCREEN时有效
- ``pygame.OPENGL`` 创建一个可以使用``opengl``的窗口
- ``pygame.RESIZABLE`` 窗口可变大小
- ``pygame.NOFRAME`` 窗口没有边框和控制条

想要实现按下<kbd>F11</kbd>全屏模式，并且窗口大小可调，可以参考以下代码

```python
import pygame, sys
from pygame.locals import *


def main():
    global screen, WINDOWWIDTH, WINDOWHEIGHT
    SIZE = WINDOWWIDTH, WINDOWHEIGHT = 1000, 800
    fps = pygame.time.Clock()
    isfullscreen = False
    pygame.init()
    screen = pygame.display.set_mode(SIZE, RESIZABLE)
    screen.fill((255,255,255)) # 背景颜色白色
    while True:
        isfullscreen = ReSize(isfullscreen)
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            if event.type == KEYDOWN:
                if event.key == K_ESCAPE:
                    pygame.quit()
                    sys.exit()
        pygame.display.update()
        screen.fill((255,255,255))
        fps.tick(30)

def ReSize(isfullscreen):
    # 这个函数中必须先判断窗口大小是否变化，在判断是否全屏
    # 否则，在全屏之后，pygame会判定为全屏操作也是改变窗体大小的一个操作，所以会显示一个比较大的窗口但不是全屏模式
    for event in pygame.event.get(VIDEORESIZE):
        size = WINDOWWIDTH, WINDOWHEIGHT = event.size[0], event.size[1]
        screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT), RESIZABLE)
    for event in pygame.event.get(KEYDOWN):
        if event.key == K_F11:
            if not isfullscreen:
                isfullscreen = True
                SIZE = WINDOWWIDTH, WINDOWHEIGHT =  pygame.display.list_modes()[0]
                screen = pygame.display.set_mode(SIZE, FULLSCREEN)
            else:
                isfullscreen = False 
                SIZE = WINDOWWIDTH, WINDOWHEIGHT = 1000, 800
                screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT), RESIZABLE)
        pygame.event.post(event)
    return isfullscreen

if __name__ == '__main__':
    main()
```

### 尚未解决的一些问题：

1. 代码中的``isfullscreen``是用来判断窗体是否是全屏，起初想要用``global``设置为全局变量，但是经过尝试之后发现怎么样都会报错，尚未找出到底是哪里错误了。

2. 一开始想要把游戏界面退出的代码也放在一个函数中，但是这样的话就发现按下<kbd>F11</kbd>之后屏幕在全屏和原来的模式之间来回闪烁，然后就闪退了，但是如果增加一个``for event in pygame.event.get()``获取事件的话，就不会有这种现象了。猜测是由于event事件没有被全部读取造成的。有无大佬指出错误，代码如下

   ```python
   def main():
       global screen, isfullscreen, WINDOWWIDTH, WINDOWHEIGHT
       # 将isfullscreen也设置为全局变量
       SIZE = WINDOWWIDTH, WINDOWHEIGHT = 1000, 800
       fps = pygame.time.Clock()
       isfullscreen = False
       pygame.init()
       screen = pygame.display.set_mode(SIZE, RESIZABLE)
       screen.fill((255,255,255)) # 背景颜色白色
       while True:
           checkQuit()
   		Resize()
           pygame.event.get() # 增加的获取事件的函数
           pygame.display.update()
           screen.fill((255,255,255))
           fps.tick(30)
   
   # 退出游戏的函数
   def terminate():
       pygame.quit()
       sys.exit()
       
   def checkQuit():
       for event in pygame.event.get(QUIT):
           terminate()
       for event in pygame.event.get(KEYDOWN):
           if event.key == K_ESCAPE:
               terminate()
           pygame.event.post(event)
   
   def Resize():
       global isfullscreen, WINDOWWIDTH, WINDOWHEIGHT, screen # 添加global声明
       for event in pygame.event.get(VIDEORESIZE):
           size = WINDOWWIDTH, WINDOWHEIGHT = event.size[0], event.size[1]
           screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT), RESIZABLE)
       for event in pygame.event.get(KEYDOWN):
           if event.key == K_F11:
               if not isfullscreen:
                   isfullscreen = True
                   SIZE = WINDOWWIDTH, WINDOWHEIGHT =  pygame.display.list_modes()[0]
                   screen = pygame.display.set_mode(SIZE, FULLSCREEN)
               else:
                   isfullscreen = False 
                   SIZE = WINDOWWIDTH, WINDOWHEIGHT = 1000, 800
                   screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT), RESIZABLE)
           pygame.event.post(event)
   
   if __name__ == '__main__':
    main()
   ```
   
   

------

### 更新

经过一段时间过后的查找，解决了第一个问题。发现需要对全局变量进行赋值的时候，需要进行声明，在函数中使用`global`声明`isfullscreen`等变量，这样进行赋值的时候才会改变全局变量的值，所以对上面的代码进行了一定的修改

关于问题二，发现只要在`Resize()`函数后增加一行获取`event`事件的函数，或者删除`Resize()`函数中的`post()`函数，就不会有闪屏退出的情况了，证明上面的猜测是正确的的，在需要获取其他事件进行相应操作的时候可以使用`post()`函数返回获取到的`KEYDOWN`事件