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
    # 是否在移动的状态
    moveup = False
    movedown = False
    moveleft = False
    moveright = False
    # 移动速度
    movespeed = 5

    def __init__(self, bgcolor, initial_pos):
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
    
    # 方向和限定区域作为参数是，限定区域是一个四元元组，(left, top, width, height),表示一个矩形区域
    def canMove(self, direction, bound):
        if direction == left and self.rect.left - self.movespeed > bound[0]:
            self.moveleft = True
            return
        elif direction == left and self.rect.left - self.movespeed <= bound[0]:
            self.rect.left = bound[0] # 如果继续移动超过了区域，那么直接移动到边界
            self.moveleft = False
            return 
        if direction == up and self.rect.top > bound[1]:
            self.moveup = True
            return
        elif direction == up and self.rect.top - self.movespeed <= bound[1]:
            self.rect.top = bound[1]
            self.moveup = False
            return     
        if direction == down and self.rect.bottom < bound[3]:
            self.movedown = True
            return
        elif direction == down and self.rect.bottom + self.movespeed >= bound[3]:
            self.rect.bottom = bound[3]
            self.movedown = False
            return 
        if direction == right and self.rect.right < bound[2]:
            self.moveright = True
        elif direction == right and self.rect.right + self.movespeed >= bound[0]:
            self.rect.right = bound[2]
            self.moveright = False
            return 
    
    # 移动函数
    def Moveleft(self):
        if self.moveleft:
            self.rect.left -= self.movespeed

    def Moveright(self):
        if self.moveright:
            self.rect.right += self.movespeed

    def Moveup(self):
        if self.moveup:
            self.rect.top -= self.movespeed

    def Movedown(self):
        if self.movedown:
            self.rect.top += self.movespeed
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

left = 'left'
up = 'up'
right = 'right'
down = 'down'

class man(pygame.sprite.Sprite):
    moveup = False
    movedown = False
    moveleft = False
    moveright = False
    movespeed = 5

    def __init__(self, bgcolor, initial_pos):
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
    
    def canMove(self, direction, bound):
        if direction == left and self.rect.left - self.movespeed > bound[0]:
            self.moveleft = True
            return
        elif direction == left and self.rect.left - self.movespeed <= bound[0]:
            self.rect.left = bound[0]
            self.moveleft = False
            return 
        if direction == up and self.rect.top > bound[1]:
            self.moveup = True
            return
        elif direction == up and self.rect.top - self.movespeed <= bound[1]:
            self.rect.top = bound[1]
            self.moveup = False
            return     
        if direction == down and self.rect.bottom < bound[3]:
            self.movedown = True
            return
        elif direction == down and self.rect.bottom + self.movespeed >= bound[3]:
            self.rect.bottom = bound[3]
            self.movedown = False
            return 
        if direction == right and self.rect.right < bound[2]:
            self.moveright = True
        elif direction == right and self.rect.right + self.movespeed >= bound[0]:
            self.rect.right = bound[2]
            self.moveright = False
            return 
    
    def Moveleft(self):
        if self.moveleft:
            self.rect.left -= self.movespeed

    def Moveright(self):
        if self.moveright:
            self.rect.right += self.movespeed

    def Moveup(self):
        if self.moveup:
            self.rect.top -= self.movespeed

    def Movedown(self):
        if self.movedown:
            self.rect.top += self.movespeed
 
def main():
    global screen
    pygame.init()
    screen = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT))
    screen.fill(WHITE)
    FPSCLOCK = pygame.time.Clock()
    Man = man(WHITE, (50,100))
    # Man.draw()
    screen.blit(Man.image, Man.rect)
    pygame.display.update()
    while True:
        checkQuit()
        # 如果想要长按一直移动，不能根据KEYDOWN事件执行操作，那样按一次按键只能动一次，所以可以使用类中的移动状态
        for event in pygame.event.get():
            if event.type == KEYDOWN:
                if event.key == K_a:
                    Man.canMove(left, BOUND)
                elif event.key == K_w:
                    Man.canMove(up, BOUND)
                elif event.key == K_s:
                    Man.canMove(down, BOUND)
                elif event.key == K_d:
                    Man.canMove(right, BOUND)
            if event.type == KEYUP:
                if event.key == K_a:
                    Man.moveleft = False
                elif event.key == K_w:
                    Man.moveup = False
                elif event.key == K_s:
                    Man.movedown = False
                elif event.key == K_d:
                    Man.moveright = False
                
        if Man.moveleft:
            Man.canMove(left, BOUND)
            Man.Moveleft()
        elif Man.moveright:
            Man.canMove(right, BOUND)
            Man.Moveright()
        if Man.moveup:
            Man.canMove(up, BOUND)
            Man.Moveup()
        elif Man.movedown:
            Man.canMove(down, BOUND)
            Man.Movedown()

        screen.fill(WHITE)
        screen.blit(Man.image, Man.rect)
        pygame.display.update()
        FPSCLOCK.tick(FPS)

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

if  __name__ == '__main__':
    main()
```

### 效果：

![效果](D:%5C%E7%94%A8%E6%88%B7%5CDesktop%5Cpython_man.gif)