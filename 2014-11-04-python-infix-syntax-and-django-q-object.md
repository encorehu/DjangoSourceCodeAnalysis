#Python中缀语法实现管道并介绍Django的Q对象

python 中缀语法? python也能管道
http://andelf.diandian.com/post/2011-03-31/40048250535

这篇文章很简单, 但是功能很强大

因为实现很简单, 所以就把代码贴过来:

    class P(object):

        def __init__(self, f):
            self.f = f

        def __ror__(self, y):
            return self.f(y)

使用示例:

    In [45]: "423423" | P(int) | P(hex) | P(str.upper)
    Out[45]: '0X675FF'

解释:

|符号在python里是"位或"操作符, 这里用一个类P来实现|操作符的重载(学过C++的人应该知道C++里的操作符的重载吧), 也就是重新实现 __ror__ 函数

上面的例子说的是 将字符串"423423" 先执行int操作, 然后再执行hex函数操作, 再大写, 也就是等价于:
str.upper(hex(int("423423")))

类似于str.upper(hex(int("423423"))) 这种语法形式的称之为前缀式语法, 也就是说语法函数这些东西在参数的前面

它执行的时候是先执行最里面的那部分, 类似与一个洋葱, 一层一层包裹, 看括号就看出来了

赖总介绍了python的管道库pipe:  http://blog.csdn.net/lanphaday/archive/2011/03/31/6291668.aspx

pipe就是管道的意思, linux中常常使用|符号作为管道, 具体管道我对linux不是特别熟悉, 所以我认为pipe类似于一个通道, 就是将前一个结果使用管道来过滤一下子.

个人觉得, 类似于将牙膏从牙膏壳中挤出来的那种, 牙膏是数据, 牙膏嘴是管道.

或者像水龙头, 水从自来水厂通过管道, 最后通过水龙头这个管道挤出来.

好了, pipe在python的简单实现介绍完了, 看完赖总的文章就知道, 已经有一个实现了很多功能的pipe库, 各位可以下载下来玩玩.

另外介绍我简单模仿刚才那个类, 做的一个过滤器吧:

    class F(object):
        '''''filter, juge every element is True or False'''

        def __init__(self, f):
            self.f = f

        def __ror__(self, seq):
            return filter(self.f, seq)

目的: 对一个list进行过滤, 先取出大于2的数值, 再取出偶数

    a=[1,2,3,4,5]
    def dayu2(x):
        return x>2

    def oushu(x):
        return x % 2==0

    #print a|F(lambda x:x>2)|F(lambda x:x % 2==0)
    print a|F(dayu2)|F(oushu)

最后得到`[4]`

这个东西仅仅对filter函数包装了一下, 用了中缀语法(即上面说的管道操作)

至于后缀语法, 我没了解, 可能是类似于这种吧:


    "123456789".replace('1','2').replace('2','3')

============================================
上面这些东西，介绍了python的中缀语法实现, 看到这里就可以看看Django中的Q对象了。

Django模型 Q对象实现复杂查询，比如and，and好实现，一个查询中只有and的话，在django中只需要filter(a=A,b=B,...)即可.

如果需要or,也就是"或", 就得需要Q对象上场了.

适用情况：执行更复杂的查询(比如，实现筛选条件的 OR、AND 关系)
例子：
models如下：

    class Article(models.Model):
        headline = models.CharField(max_length=50)
        pub_date = models.DateTimeField()

想查询headline字段开头以'hello'，或者结尾以'Goodbye'的数据。
代码实现：

    Article.objects.filter(Q(headline__startswith='Hello') | Q(headline__startswith='Goodbye'))

AND的实例：

    Article.objects.filter(~Q(pk=self.a1) & ~Q(pk=self.a2))

看到 "|"和"&" 这俩符号了吧, 前一个是or,后一个是and, 但是是使用中缀语法来表示的, 结合上面的介绍, 可以到Django的Q对象实现的源码中去看看:

    # 位于/django/db/models/query_utils.py

    class Q(tree.Node):
        """
        Encapsulates filters as objects that can then be combined logically (using
        & and |).
        """
        # Connection types
        AND = 'AND'
        OR = 'OR'
        default = AND

        def __init__(self, *args, **kwargs):
            super(Q, self).__init__(children=list(args) + kwargs.items())

        def _combine(self, other, conn):
            if not isinstance(other, Q):
                raise TypeError(other)
            obj = type(self)()
            obj.add(self, conn)
            obj.add(other, conn)
            return obj

        def __or__(self, other):
            return self._combine(other, self.OR)

        def __and__(self, other):
            return self._combine(other, self.AND)

        def __invert__(self):
            obj = type(self)()
            obj.add(self, self.AND)
            obj.negate()
            return obj

看到了吧, 重新定义了__or__, __and__, 和__invert__, 从而实现了Q对象的管道操作, 使用的是中缀语法.