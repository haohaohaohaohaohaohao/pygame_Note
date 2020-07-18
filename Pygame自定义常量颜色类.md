# ``Pygame``如何利用常量类自定义颜色

在使用``pygame``的时候，经常会用到一些颜色的RGB值，我们可以在程序一开始进行自定义，如：

```python
BLACK = (  0,  0,  0)
WHITE = (255,255,255)
```

但是我们每个程序都这样操作就会显得有点重复，如果一个项目由几个文件组成，在每个文件里就都需要重新自定义颜色的RGB值了，这个时候，我们就可以利用``sys.modules``来实现常量类，将颜色的RGB值设置为常量储存在一个文件中，这样我们使用的时候就会更加方便。

### 实现：

- ``const.py``文件：自定义常量类

    ```python
    # const.py
    class _const:
        class ConstError(TypeError):
            pass
    
        class ConstCaseError(ConstError):
            pass
    
        def __setattr__(self, name, value):
            if name in self.__dict__:
                raise self.ConstError("Can't change const.{}".format(name))
            if not name.isupper():
                raise self.ConstCaseError("const name {} is not all uppercase".format(name))
            self.__dict__[name] = value
    
    import sys
    sys.modules[__name__] = _const()
    ```

- ``color.py``文件：存储颜色的RGB值

    ```python
    # color.py
    import const
    
    const.BLACK = (0, 0, 0)
    const.WHITE = (255,255,255)
    ```

- ``test.py``文件：测试文件

    ```python
    from color import const
    
    print(const.BLACK)
    print(const.WHITE)
    ```

- 结果：

    ```
    python test.py
    (0, 0, 0)
    (255, 255, 255)
    ```

    ### 原理：

    首先，在运行`test.py`文件的时候，先运行`from color import const`，关于`from module import...`在某乎看到了非常有意思的比喻，认为也同样适用在这里。博主是这样理解的，在`color.py`中`import const`相当于把`const`这个篮子放进了`color`这一个大篮子里，然后把`color`中的这个`const`篮子里放入一些东西，`from color import const`就是将`color`中的装了东西的`const`篮子拿出来使用，所以在`test.py`文件中使用`const.BLACK`就可以了。

    其次，关于`const.py`的原理 ，`sys.modules[name] = _const()`这一行代码手动的将`constant`写入`sys.modules`，然后当`const.py`load结束之后，回到了`color.py`中的`import`位置，程序尝试将`const`写入`sys.modules`，可是此时`constant`已经在`sys.modules`中了，所以不会再次写入，这样在`color.py`中`import constant`时，实际上获取到的是`_const()`
    
	具体原理可参考博客：
	[python将常量集中在一个文件中、常量类的实现、sys.modules的使用](https://blog.csdn.net/u010080235/article/details/100042727?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
	[Python中实现常量（Const）功能](https://blog.csdn.net/lushujuan88/article/details/43236181?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)
	
	
