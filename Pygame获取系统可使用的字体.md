# `pygame`如何获取系统可使用的字体

`pygame.font`模块中有一个`pygame.font.get_fonts()`函数，返回值为一个列表，包含了系统中所有的可以使用的字体，可以用这个函数将所有的可用的字体都输出到一个文件之中，方便查找：

```python
import pygame

fonts = pygame.font.get_fonts()

with open('font.txt', 'w', encoding = 'utf-8') as file:
    for font in fonts:
    	file.write(font)
        file.write('\n')
```

### 结果：![image-20200718104754642](D:%5C%E7%94%A8%E6%88%B7%5CDesktop%5Cfont.png)