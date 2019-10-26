## neo4j

### 增：

\#  创建一个节点

create  (p:pig {name:"猪爷爷"} )  return p



\#  同时创建多个节点 

create  (:pig {name:"猪爷爷",age:6} ) , (:pig {name:"猪奶奶",age:4} ) , (:pig {name:"猪爸爸",age:3} ) , (:pig {name:"猪妈妈",age:2} )



\#  创建关系  先查询出需要增加关系的节点  分别赋值  再给这两个节点增加关系

```python


match (gf:pig{name:"猪爷爷"}) match (gm:pig{name:"猪奶奶"}) match (f:pig{name:"猪爸爸"}) match (m:pig{name:"猪妈妈"}) create (gf)-[r:夫妻]-(gm) create (gf)-[gx:公媳]-(m)  create (gm)-[px:婆媳]-(m)   return gf,r,gm

 
```

## 改

先将节点查出来  将属性设置好，然后返回设置的属性

```python
MATCH (n:pig{name:"佩奇"}) set n.age=0.6 return n
```

## 删

```python
MATCH (n:pig{name:"佩奇"}) delete n;
```

```yacas

MATCH (n:pig{name:"佩奇"}) where n.age=0.6 delete n;

(n:pig{name:"佩奇"}) where n.age=0.6  这些表示条件


与其他节点存在关系时不能直接删除 需要和关系一起删掉
MATCH (n.pig{name:"佩奇"})-[r]-() where n.age=0.6 delete r,n;

[] 代表任意关系
() 代表任意节点
delete r,n; 表示将关系和节点全部删掉
```







