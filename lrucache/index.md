# Leetcode - 146. LRU Cache


Leetcode - 146. LRU Cache

<!--more-->

## Least Recently Used Cache

### Cache?

cache는 데이터나 값을 미리 복사해 놓는 임시 장소를 가리킨다.  
접근 시간에 비해 원래 데이터를 접근하는 시간이 오래걸리는 경우, 다시 값을 계산하는시간을 절약 하고 싶을때 사용한다.  
캐시에 미리 데이터를 복사 해놓으면 빠른 속도로 데이터접근이 가능하다.

### LRU Cache

LRU는 OS의 페이지 교체 알고리즘중 하나이다.  
가장 오랫동안 사용하지 않은 페이지를 교체하는 기법이다.

### LRU Cache 구현

LRU Cache는 head에 가까운 데이터일수록 최근에 사용한 데이터이다.
tail은 가까울 수록 오랫동안 사용하지 않은 데이터이다. 또한 새로운 데이터를 삽입할때 가장 먼저 삭제되도록 한다.

#### code

~~~go
type LRUCache struct {
    capacity int
    cache map[int]*node
    head, tail *node
}

type node struct {
    prev, next *node
    key, value int
}
~~~
LRUCache를 정의하고 node를 linked list로 묶기위해서 node를 정의한다.

~~~go
func Constructor(capacity int) LRUCache {
    return LRUCache{
        capacity: capacity,
        cache: map[int]*node{},
        head: nil, 
        tail: nil,
    }
}
~~~
초기화 시키는 함수

~~~go
func (this *LRUCache) Get(key int) int {
    node, ok := this.cache[key]
    
    if !ok {
        return -1
    }
    
    if this.head != nil && node.key == this.head.key {
        return node.value
    }

if this.tail != nil && node.key == this.tail.key {
        this.pop()
        this.insert(node)
        this.cache[node.key] = node
        return node.value
    }

if node.prev != nil && node.next != nil {
        node.prev.next = node.next
        node.next.prev = node.prev
        this.insert(node)
    }
    
    return node.value
}
~~~
Get 함수이다. 데이터를 가져오는데 Cache의 특성상 사용(Get)한 데이터는 head쪽에 업데이트를 해주기 위해서 pop하고 insert를 다시해주는걸 볼 수 있다.

~~~go
func (this *LRUCache) Put(key int, value int)  {
    if this.Get(key) != -1 {
        this.cache[key].value = value
        return
    }
    
    n := &node{key: key, value: value}
    this.cache[key] = n
    this.insert(n)
    
    if len(this.cache) > this.capacity {
        this.pop()
    }
    return  
}
~~~
Put 함수이다. 데이터를 넣는데 사용한다. cache크기를 넘어서면 pop으로 tail의 데이터를 제거해준다.

~~~go
func (this *LRUCache) insert(n *node) {
    if this.tail == nil {
        this.tail = n
        this.head = n
        n.next = this.tail
        return
    }
    this.head.prev = n
    n.next = this.head
    this.head = n
}
~~~
insert를 하면 head를 업데이트 해준다.

~~~go

func (this *LRUCache) pop() {
    delete(this.cache, this.tail.key)
    if this.tail.prev == nil {
        this.tail = nil
        return
    }
    this.tail.prev.next = nil
    this.tail = this.tail.prev
}
~~~
tail의 데이터를 꺼낸다.
