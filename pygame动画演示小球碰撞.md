# 使用pygame实现动量定理的小球碰撞演示动画

动量定理我们在高中的时候就已经接触过了，是十分重要的物理定理。其中的完全弹性碰撞（机械能守恒）是十分典型的例子，机械能守恒和动量定理两个公式就可以推出小球碰撞之后的速度，依据这个公式，我们就可以完成我们的动画演示了。
$$
\begin{equation}
m_1v_1+m_2v_2=m_1v_1'+m_2v_2'
\end{equation}
$$

$$
\begin{equation}
\frac{1}{2}m_1v_1^2+\frac{1}{2}m_2v_2^2=\frac{1}{2}m_1v_1'^2+\frac{1}{2}m_2v_2'^2
\end{equation}
$$



根据这两个式子，可以得出碰撞之后的$v_1’$和$v_2'$的速度：
$$
\begin{equation}
v_1'=\frac{2m_2}{m_1+m_2}v_2+\frac{m_1-m_2}{m_1+m_2}v_1
\end{equation}
$$

$$
\begin{equation}
v_2'=\frac{2m_1}{m_1+m_2}v_1+\frac{m_2-m_1}{m_1+m_2}v_2
\end{equation}
$$

然后就可以写代码了

## 源代码

```python
# color.py
# 需要用到的颜色
RED = (255,0,0)
GREEN = (0,255,0)
BLUE = (0,0,255)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
YELLOW = (255,255,0)
UNKNOWN = (0, 255,255)
```

```python
import pygame
from pygame.locals import *
from color import *

# 精灵类
class ball(pygame.sprite.Sprite):
    def __init__(self, mass, position, velocity, color): # position 为左下角的坐标
        self.mass = mass
        self.image = pygame.Surface((40, 40))
        self.rect = self.image.get_rect()
        # pygame.draw.rect(self.image, BLACK, self.rect, 1)
        self.image.fill(WHITE)
        pygame.draw.circle(self.image, color, (20, 20), 20)
        self.rect.bottomleft = position
        self.velocity = velocity
```

```python
# 主函数
import pygame, sys
from pygame.locals import *
from color import *
from ball import ball

def main():
    pygame.init()
    screen = pygame.display.set_mode((800, 600))
    pygame.display.set_caption('动量定理动画')
    FPSCLOCK = pygame.time.Clock()
    floor1 = 200
    floor2 = 400
    floor3 = 600
    # 初始化小球
    ball1 = ball(10, (0, floor1), 10,RED)
    ball2 = ball(20, (760, floor1), -5,BLACK)
    ball3 = ball(10, (0, floor2), 10,GREEN)
    ball4 = ball(10, (760, floor2), -10,BLUE)
    ball5 = ball(1, (0, floor3), 10, YELLOW)
    ball6 = ball(1000000000000000000000, (400, floor3), 0, UNKNOWN)
    # 绘制两个小球
    screen.fill(WHITE)
    pygame.draw.line(screen, BLACK,(0, floor1), (800, floor1), 1)
    pygame.draw.line(screen, BLACK,(0, floor2), (800, floor2), 1)
    pygame.draw.line(screen, BLACK,(0, floor3), (800, floor3), 1)
    screen.blit(ball1.image, ball1.rect)
    screen.blit(ball2.image, ball2.rect)
    screen.blit(ball3.image, ball3.rect)
    screen.blit(ball4.image, ball4.rect)
    screen.blit(ball5.image, ball5.rect)
    screen.blit(ball6.image, ball6.rect)

    while True:
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()

        if collision_check(ball1.rect, ball2.rect):
            v1 = ball1.velocity
            v2 = ball2.velocity
            m1 = ball1.mass
            m2 = ball2.mass
            ball1.velocity = int(2*m2*v2/(m1+m2)+((m1-m2)/(m1+m2))*v1)
            ball2.velocity = int(2*m1*v1/(m1+m2)+((m2-m1)/(m1+m2))*v2)

        if ball3.rect.right > ball4.rect.left and ball3.rect.right - ball3.velocity <= ball4.rect.left - ball4.velocity:
            v1 = ball3.velocity
            v2 = ball4.velocity
            m1 = ball3.mass
            m2 = ball4.mass
            ball3.velocity = int(2*m2*v2/(m1+m2)+((m1-m2)/(m1+m2))*v1)
            ball4.velocity = int(2*m1*v1/(m1+m2)+((m2-m1)/(m1+m2))*v2)

        if ball5.rect.right > ball6.rect.left and ball5.rect.right - ball5.velocity <= ball6.rect.left - ball6.velocity:
            v1 = ball5.velocity
            v2 = ball6.velocity
            m1 = ball5.mass
            m2 = ball6.mass
            print(ball5.velocity)
            ball5.velocity = int(2*m2*v2/(m1+m2)+((m1-m2)/(m1+m2))*v1)
            ball6.velocity = int(2*m1*v1/(m1+m2)+((m2-m1)/(m1+m2))*v2)

        if ball1.rect.left < 0 or ball1.rect.right > 800:
            ball1.velocity = -ball1.velocity
            if (ball1.rect.left < 0 and ball1.velocity < 0) or (ball1.rect.right > 800 and ball1.velocity > 0):
                ball1.velocity = -ball1.velocity

        if ball2.rect.left < 0 or ball2.rect.right > 800:
            ball2.velocity = -ball2.velocity
            if (ball2.rect.left < 0 and ball2.velocity < 0) or (ball2.rect.right > 800 and ball2.velocity > 0):
                ball2.velocity = -ball2.velocity

        if ball3.rect.left < 0 or ball3.rect.right > 800:
            ball3.velocity = -ball3.velocity
            if (ball3.rect.left < 0 and ball3.velocity < 0) or (ball3.rect.right > 800 and ball3.velocity > 0):
                ball3.velocity = -ball3.velocity

        if ball4.rect.left < 0 or ball4.rect.right > 800:
            ball4.velocity = -ball4.velocity
            if (ball4.rect.left < 0 and ball4.velocity < 0) or (ball4.rect.right > 800 and ball4.velocity > 0):
                ball4.velocity = -ball4.velocity

        if ball5.rect.left < 0 or ball5.rect.right > 800:
            ball5.velocity = -ball5.velocity
            if (ball5.rect.left < 0 and ball5.velocity < 0) or (ball5.rect.right > 800 and ball5.velocity > 0):
                ball5.velocity = -ball5.velocity

        if ball6.rect.left < 0 or ball6.rect.right > 800:
            ball6.velocity = -ball6.velocity
            if (ball6.rect.left < 0 and ball6.velocity < 0) or (ball6.rect.right > 800 and ball6.velocity > 0):
                ball6.velocity = -ball6.velocity

        ball1.rect.left += ball1.velocity
        ball2.rect.left += ball2.velocity
        ball3.rect.left += ball3.velocity
        ball4.rect.left += ball4.velocity
        ball5.rect.left += ball5.velocity
        ball6.rect.left += ball6.velocity

        screen.fill(WHITE)
        pygame.draw.line(screen, BLACK,(0, floor1), (800, floor1), 1)
        pygame.draw.line(screen, BLACK,(0, floor2), (800, floor2), 1)
        pygame.draw.line(screen, BLACK,(0, floor3), (800, floor3), 1)
        screen.blit(ball1.image, ball1.rect)
        screen.blit(ball2.image, ball2.rect)
        screen.blit(ball3.image, ball3.rect)
        screen.blit(ball4.image, ball4.rect)
        screen.blit(ball5.image, ball5.rect)
        screen.blit(ball6.image, ball6.rect)
        pygame.display.update()
        FPSCLOCK.tick(60)

if __name__ == '__main__':
    main()
```

### 效果

![](D:%5C%E7%94%A8%E6%88%B7%5CDesktop%5C%E6%A1%8C%E9%9D%A2%E4%B8%B4%E6%97%B6%E6%96%87%E4%BB%B6%5Cmomentum.gif)