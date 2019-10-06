# 2019-09-30 用Python实现简单电影推荐(dict的练习)



> 关于python的容器，印象最深刻的就是字典，dict和java的map极为相似，但是在用法上面有一些不同，现在将我做过的练习Po出来，作为一个记录

## 问题提出

![image-20190930203630528](/Users/apple/Desktop/MarkDown笔记/文章图片/image-20190930203630528.png)

**题目来自董付国老师的python小屋ppt，侵删**



***

**总结一下这个问题的关键——如何给一个用户推荐一部电影**

- 找到和这个被测用户口味相同的其他用户，也就是说，其他用户必须和这个用户看过的电影的交集尽可能大，当然这种情况很可能挑出不止一个用户
- 在上面找到的这些用户里面，选出和这个被测用户，打分最相似的。也就是说，最后挑选出来的用户，不仅要看过被测用户看过的电影，喜爱程度(打分)还要相近。
- 最后挑出一个最相近的用户，在这个用户看过，但是被测用户没看过的集合里面，挑选出一部电影来给这个用户



**到此，已经明白了我们需要做什么**

- 要设计算法，求出看过电影交集的最大的用户的集合
- 设计算法，求出打分最相近的用户的集合
- 设计算法，在最相似的用户没看过的电影里面挑出没看过，但是打分最高的电影



## 知识点大纲

### 字典是什么

1. 可变容器，存储任意可变对象
2. key唯一，value不唯一，思维从java的map迁移即可
3. 使用items()方法将字典每一项转化tuple，可用下标去访问。
4. 使用keys()方法作用在dict上，可以获取这个dict的所有key

### 怎么求字典交集

**如下，使用 & 操作符即可**

```python
data1 = {'movie1':9,'movie3':10,'movie5':4}
data2 = {'movie1':4,'movie2':3,'movie5':4}

print(data1.keys()&data2.keys())
```

在python3里面，你甚至可以第三行其中任意一个写成`print(data1&data2.keys())`，也是允许的，这里求取的就是两个字典的key的交集。**最后输出**

`{'movie1', 'movie5'}`



### 怎么求最相似的打分

**首先观察原始的数据结构**

```python
data1 = {'movie1':9,'movie3':10,'movie5':4}
data2 = {'movie1':4,'movie2':3,'movie5':4}
```

这种样子的数据结构，和n维空间的向量非常相似，这种比较向量的距离来作为向量的相似度的做法非常常见。

我们可以使用求两个向量之间的方差，方差越小，说明两者的电影口味相似度高。

```python
f = lambda item : sum((item.get(i)-data2.get(i))**2 for i in item.keys() & data2.keys())
num = f(data1)
```

比如上面代码，就可求出上面两组数据，相同key之间（movie1和movie5）的方差，为**25**。



***

### 怎么求差集

当我们知道哪个用户和被测用户最相似之后，我们需要在这个用户看过的其他电影中选取分数最高的。

**提取关键字，我们可以提炼出两条信息**

- 这个用户看过的其他电影，也即是这个用户和被测用户的差集
- 分数最高

**到这里，我们可以自然的想到做法**

max函数，但是自定义一个rule，将key的规则变成 获取这个dict的value，也就是dict[key]



***

### 其他重要知识点

#### lambda函数

匿名函数、lambda关键字后面跟着的是参数，：后面跟着的是想要返回的东西，可能是整数，可能是容器...

常和 filter以及max/min函数配合服用。



#### filter函数

过滤器，我理解为剔除器，将列表，以及保留规则（函数或者lambda函数）传入，即可处理一个容器集合。将符合保留条件的保留下来。



#### max函数

求最大值，这里要讲的不是简单意义上的max函数，而是自定义的max函数。

自定义max函数需要补充key属性，一般是个函数或者lambda。自定义max函数传的两个参数中：第一个是集合，也就是返回的是该集合中的元素；第二个我理解为元素映射的值的指示函数，自定义max函数会根据这个key里面的函数，获取前面第一个参数中元素对应的值，这样才能比较大小。



> 在字典里面，我们通常需要返回key，但是只能比较value。上面这种自定义的max函数完美的符合这种需求





***

## 董教授的代码

![image-20191002141001093](/Users/apple/Desktop/MarkDown笔记/文章图片/image-20191002141001093.png)

董教授的代码真的很pythonic，尤其是红线画出来的地方。

将求最多相似电影和最小方差评分在一个lambda函数里，最后用一个min函数就完成了最相似用户的求取。**将求最大值转化为求最小值加负号的思想真的很值得学习**。

但是个人觉得对于我这种正在学习阶段的人来说，这种代码在可理解性上差了点，小技巧以后自然会习得，但是当务之急是熟练这些高级函数和容器的使用。



## 思考和改进思路

**那么，我将怎么组织我的代码？**

1. 生成用户数据，以及被测用户数据，全部随机生成，使用random库
2. 先求最多相似的用户，将每个用户和被测用户的重复的key计数，求出最大重复数
3. 使用filter，将用户数据里面符合最大重复数的用户保留，其余剔除
4. 在新的用户数据里面求出最小的方差，将那个最小方差的用户提取出来
5. 在这个提取出来的用户看过的电影中，找出被测用户没看过，并且这个用户评分最高的电影
6. 结束

**其中，2-3-4是我讲董教授的代码步骤进行拆分的结果**



## 我的代码

```python
from random import randint,randrange

user_data={}

for i in range(10):
    user_data['user'+str(i)]={}
    # 接下来给每个用户随机生成电影20次，每次可能和之前生成的某次电影编号相似
    for movie in range(10):
        # 给user0-user9，每个人看过的每个电影打上1-10分
        user_data['user'+str(i)]['movie'+str(randrange(3,9))] = randrange(1,11)
print('所有用户数据'.center(100,'='))
for i in user_data.items():
    print(i)
#########################################################################################
print('测试用户数据'.center(100,'='))
test_user = {'movie'+str(randrange(3,9)):randrange(1,11) for movie in range(5)}
print(test_user)

#########################################################################################
same_movie = [len(i[1].keys() & test_user.keys())for i in user_data.items()]
max_same_number = max(same_movie)
print('='.center(100,'='))
print("每个用户和被测用户看过一样的电影数量"+str(same_movie)+"      用户最多看过 "+str(max_same_number)+" 部和被测用户一样的电影")
#########################################################################################
print('最多电影相似用户（处理后数据）'.center(100,'='))

flt_func = lambda item:len(item[1].keys()&test_user)==max_same_number

flt_func1 = lambda new_user_data:sum(((new_user_data[1].get(movie)-test_user.get(movie))**2 for movie in new_user_data[1].keys()&test_user))


new_dict = filter(flt_func,user_data.items())

new_user_data = dict(new_dict)

for i in new_user_data.items():
    print(i)
#########################################################################################
print('最小欧拉距离'.center(100,'='))

most_sim_user = min(new_user_data.items(),key=flt_func1)

print(most_sim_user)
#########################################################################################
print('最后推荐的电影是'.center(100,'='))
print(max(most_sim_user[1].keys()-test_user.keys(),key = lambda item:most_sim_user[1][item]))
```

全部代码共计50行以内，写的比较啰嗦，并且变量和函数命名比较丑陋。

以后的学习会多注意多参考。