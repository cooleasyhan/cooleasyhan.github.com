title: 无向图
date: 2015-12-02
tags: Algorithms, graph
slug: graph


## 基本概念

图是由一组顶点(vertex)和一组能够将两个顶点相连的边(edge)组成的  

图种类： 自环，平行边， 简单图  
这里主要关注简单图
### 术语表
术语|含义
----|----
相邻|
依附|
度数|
子图|识别子图是核心问题之一，尤其是那些能够顺序连接一系列顶点的边所组成的子图
路径|
简单路径|一条没有重复顶点的路径
环|
简单环|
路径，环的长度|
连通| 两个顶点存在路径
连通图|
极大连通子图|
无环图|
树|
森林|
生成树|
生成树森林|
稠密图|
稀疏图|
二分图|

### 一颗树条件
一幅含有V个节点的图G满足下列5个条件之一，它就是一棵树  

	- G 有V-1条边且不含有环
	- G 有V-1条边且是连通的
	- G 是连通的， 但删除任意一条边度会使它不再联通
	- G 是无环图， 但添加任何一条边都会产生一个环
	- G 中的任意一对顶点之间仅存在一条简单路径



## API
为了隔离图的表示和实现，我们需要一个数据类Graph， 和一堆算法实现类。算法实现类会在构造函数里接收Graph，并做出相应的数据处理，提供外部接口供外部使用，响应用例的请求  

Class Graph，一般包含构造函数，确定节点个数；添加边，获取顶点数和边数，获取某个顶点相邻的节点

方法|描述
----|----
Graph(int v) | 构造函数, 创建一个含有V个顶点但是不含有边的图
V() | 顶点数
E() | 边数
void addEdge(int v, int w) | 添加一条边
Iterable adj(int v) | 和v 相邻的所有顶点
String toString() 

## 图的构件方法
- 邻接矩阵  
V*V 的布尔矩阵
- 边的数组  
- 邻接表数组  
有些像使用链表的hashtable，以顶点为索引做成一个数组，数组每个值为与该顶点相邻的顶点列表。 相邻顶点可以使用链表或者集合实现。链表允许平行边  
空间复杂度： E+V  
添加一条边：1  
检查两节点是否相邻 degree(v)  
便利v的所有相邻节点 degree(v)  


## 深度优先搜索
逻辑： 选取一个顶点，按顺序获取他的相邻节点，找到任何一个未访问的节点，则检索该未访问节点的相邻节点，如果所有相邻节点均已经被访问，则返回，将每一个访问过的节点存储至两个一维数组，marked数组用于判定该顶点是否被访问到。 edge_to数组存储的是其前一个顶点。比如v-w 则存储为edge_to[w] = v  
获取路径则是从终点回退到起点，将路径上的节点均压入栈，再顺序取出



```
#!/bin/python
# -*- coding: UTF-8 -*-

'''
Author: YiHan
Date: 2015-12-03
无向图API
'''


class Graph(object):

    '''
    创建一个含有V个顶点但是不含有边的图
    '''

    def __init__(self, v):
        self._V = v
        self._E = 0
        self.adj_list = list()
        for i in range(v):
            self.adj_list.append(list())

    def V(self):
        '''
        顶点数
        '''
        return self._V

    def E(self):
        '''
        边数
        '''
        return self.E

    def add_edge(self, v, w):
        '''
        添加边
        '''
        self.adj_list[v].append(w)
        self.adj_list[w].append(v)
        self._E += 1

    def adj(self, v):
        '''
        和顶点v相邻的所有顶点
        '''
        return self.adj_list[v]

    def to_string(self):
        print self._V, 'vertices,', self._E, 'edges'
        for idx, val in enumerate(self.adj_list):
            s = str(idx)
            for v in self.adj_list[idx]:
                s = s + ' ' + str(v)
            print s


class DepthFirstPaths(object):
    '''
    查找所有与其相连的节点
    '''

    def __init__(self, G, s):
        self.marked_list = list()
        self._count = 0
        self._G = G
        self._s = s
        self.edge_to = list()
        for i in range(G.V()):
            self.marked_list.append(False)
            self.edge_to.append(None)
        self.dfs(s)

    def dfs(self, v):

        self.marked_list[v] = True
        self._count += 1

        for w in self._G.adj(v):
            if not self.marked(w):
                self.edge_to[w] = v
                self.dfs(w)

    def marked(self, v):
        return self.marked_list[v]

    def count(self):
        return self._count

    def has_path_to(self, v):
        return self.marked_list[v]

    def path_to(self, v):
        if not self.has_path_to(v):
            return None

        path = list()
        pre = v
        while pre != self._s:
            path.insert(0, pre)
            pre = self.edge_to[pre]

        path.insert(0, self._s)
        return path

def main():
    g = Graph(13)
    g.add_edge(0, 1)
    g.add_edge(0, 2)
    g.add_edge(0, 6)
    g.add_edge(0, 5)
    g.add_edge(5, 4)
    g.add_edge(5, 4)
    g.add_edge(4, 6)
    g.add_edge(7, 8)
    g.add_edge(9, 10)
    g.add_edge(9, 11)
    g.add_edge(9, 12)
    g.add_edge(11, 12)
    g.add_edge(3, 4)
    g.add_edge(6, 9)
    g.to_string()

    ds = DepthFirstPaths(g, 1)
    rst = ""
    for i in range(g.V()):
        if ds.marked(i):
            rst = rst + str(i) + " "

    print rst
    print 'count:', ds.count()

    print ds.has_path_to(12)

    print ds.path_to(12)

if __name__ == '__main__':
    main()
```

