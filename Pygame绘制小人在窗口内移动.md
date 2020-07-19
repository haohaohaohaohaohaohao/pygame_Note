# ``Pygame``如何绘制一个小人在窗口内移动

利用·``pygame.draw``模块可以绘制一些简单的图像，但是我们怎样可以实现利用键盘控制图像使它移动呢？

###　函数实现

首先，图像的移动本质其实就是点的移动。可以根据一个参考点绘制出相对应的图像，改变参考点的坐标就可以实现图像的整体移动。

```python
# 传入左上角的位置坐标
BLACK = (0, 0, 0)
def drawMan(left, top):
    pygame.draw.circle(screen, BLACK, (left+50, top+30), 30, 2)
    pygame.draw.line(screen, BLACK, (left+50, top+60), (left+50, yop+140), 2)
    pygame.draw.line(screen, BLACK, (left+50, top+140), (left, top+200), 2)
    pygame.draw.line(screen, BLACK, (left+50, top+140), (left+100, top+200), 2)
    pygame.draw.line(screen, BLACK, (left+50, top+100), (left, top+60), 2)
    pygame.draw.line(screen, BLACK, (left+50, top+100), (left+100, top+60), 2)
```

### 类实现

利用以上函数就可以通过控制左上角的位置坐标来实现小人的移动，但是这样要进行其他的操作就比较困难，那么如果我们可以把这个图像绘制在另外一个``Surface``对象上，那么就会比较容易一些。这个时候我们就可以使用``pygame``中的``pygame.sprite.Sprite``精灵类，精灵可以看作是一个屏幕上的图形对象，并且可以实现移动等一系列操作。如何使用可以参考官方文档：[官方文档链接](http://www.pygame.org/docs/ref/sprite.html#pygame.sprite.Sprite)

最简单的一个继承``pygame.sprite.Sprite``精灵类的实现方法（来源于官方文档）：

```python
class Block(pygame.sprite.Sprite):
    # Constructor. Pass in the color of the block,
    # and its x and y position
    def __init__(self, color, width, height):
       # Call the parent class (Sprite) constructor
       pygame.sprite.Sprite.__init__(self)

       # Create an image of the block, and fill it with a color.
       # This could also be an image loaded from the disk.
       self.image = pygame.Surface([width, height])
       self.image.fill(color)

       # Fetch the rectangle object that has the dimensions of the image
       # Update the position of this object by setting the values of rect.x and rect.y
       self.rect = self.image.get_rect()
```

模仿这样的模式可以将我们需要的类创建出来：

```python
class man(pygame.sprite.Sprite):
    # 移动速度
    movespeed = 5

    def __init__(self, bgcolor, initial_pos, bound):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.Surface((100, 200))
        self.image.fill(bgcolor)
        pygame.draw.circle(self.image, BLACK, (50, 30), 30, 2)
        pygame.draw.line(self.image, BLACK, (50, 60), (50, 140), 2)
        pygame.draw.line(self.image, BLACK, (50, 140), (0, 200), 2)
        pygame.draw.line(self.image, BLACK, (50, 140), (100, 200), 2)
        pygame.draw.line(self.image, BLACK, (50, 100), (0, 60), 2)
        pygame.draw.line(self.image, BLACK, (50, 100), (100, 60), 2)
        self.rect = self.image.get_rect()
        self.rect.topleft = initial_pos
        self.bound = bound
        # 表示小人移动的范围，四元列表，分别表示左边界，右边界，上边界，下边界
        
    # 移动函数
    def Moveleft(self):
        if  self.rect.left - self.movespeed >= self.bound[0]:
            self.rect.left -= self.movespeed
        elif self.rect.left - self.movespeed < self.bound[0]:
            self.rect.left = self.bound[0]

    def Moveright(self):
        if self.rect.right + self.movespeed <= self.bound[1]:
            self.rect.right += self.movespeed
        elif self.rect.right + self.movespeed > self.bound[1]:
            self.rect.right = self.bound[1]
            
    def Moveup(self):
        if  self.rect.top - self.movespeed >= self.bound[2]:
            self.rect.top -= self.movespeed
        elif self.rect.top - self.movespeed < self.bound[2]:
            self.rect.top = self.bound[2]

    def Movedown(self):
        if  self.rect.bottom + self.movespeed <= self.bound[3]:
            self.rect.bottom += self.movespeed
        elif self.rect.bottom + self.movespeed > self.bound[3]:
            self.rect.bottom = self.bound[3]
```

### 完整代码：

```python 
import pygame, sys
from pygame.locals import *

FPS = 60
WINDOWWIDTH = 1000
WINDOWHEIGHT = 800
SIZE = (WINDOWWIDTH, WINDOWHEIGHT)
BOUND = (0, 0, WINDOWWIDTH, WINDOWHEIGHT)
WHITE = (255, 255, 255)
BLACK = (0, 0 ,0)
RED = (255,0,0)

class man(pygame.sprite.Sprite):
    movespeed = 5

    def __init__(self, bgcolor, initial_pos, bound):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.Surface((100, 200))
        self.image.fill(bgcolor)
        pygame.draw.circle(self.image, BLACK, (50, 30), 30, 2)
        pygame.draw.line(self.image, BLACK, (50, 60), (50, 140), 2)
        pygame.draw.line(self.image, BLACK, (50, 140), (0, 200), 2)
        pygame.draw.line(self.image, BLACK, (50, 140), (100, 200), 2)
        pygame.draw.line(self.image, BLACK, (50, 100), (0, 60), 2)
        pygame.draw.line(self.image, BLACK, (50, 100), (100, 60), 2)
        self.rect = self.image.get_rect()
        self.rect.topleft = initial_pos
        self.bound = bound
        # 表示小人移动的范围，四元列表，分别表示左边界，右边界，上边界，下边界
    
    def Moveleft(self):
        if  self.rect.left - self.movespeed >= self.bound[0]:
            self.rect.left -= self.movespeed
        elif self.rect.left - self.movespeed < self.bound[0]:
            self.rect.left = self.bound[0]

    def Moveright(self):
        if self.rect.right + self.movespeed <= self.bound[1]:
            self.rect.right += self.movespeed
        elif self.rect.right + self.movespeed > self.bound[1]:
            self.rect.right = self.bound[1]
            
    def Moveup(self):
        if  self.rect.top - self.movespeed >= self.bound[2]:
            self.rect.top -= self.movespeed
        elif self.rect.top - self.movespeed < self.bound[2]:
            self.rect.top = self.bound[2]

    def Movedown(self):
        if  self.rect.bottom + self.movespeed <= self.bound[3]:
            self.rect.bottom += self.movespeed
        elif self.rect.bottom + self.movespeed > self.bound[3]:
            self.rect.bottom = self.bound[3]
 
def main():
    global screen
    pygame.init()
    screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT))
    screen.fill(WHITE)
    FPSCLOCK = pygame.time.Clock()
    Man = man(WHITE, (50,100), (0, WINDOWWIDTH, 0, WINDOWHEIGHT))
    screen.blit(Man.image, Man.rect)
    pygame.display.update()
    while True:
        # 如果想要长按一直移动，不能根据KEYDOWN事件执行操作，那样按一次按键只能动一次
        # 可以使用pygame中的key.get_pressed()函数
        keys = pygame.key.get_pressed()

        for event in pygame.event.get():
            if event.type == QUIT:
                terminate()

        if keys[K_ESCAPE]:
            terminate()
        if keys[K_a] or keys[K_LEFT]:
            Man.Moveleft()
        if keys[K_d] or keys[K_RIGHT]:
            Man.Moveright()
        if keys[K_s] or keys[K_DOWN]:
            Man.Movedown()
        if keys[K_w] or keys[K_UP]:
            Man.Moveup()
                

        screen.fill(WHITE)
        screen.blit(Man.image, Man.rect)
        pygame.display.update()
        FPSCLOCK.tick(FPS)

def terminate():
    pygame.quit()
    sys.exit()

if  __name__ == '__main__':
    main()
```

### 效果：

![效果](D:%5C%E7%94%A8%E6%88%B7%5CDesktop%5Cpython_man.gif)