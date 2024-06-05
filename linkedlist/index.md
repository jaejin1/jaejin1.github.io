# Python3에서 Linked List에 관한 여러 연산들


python3에서 Linked List에 대한 여러 연산들을 정리 해놓자.

안하면 자꾸 까먹어서..

<!--more-->

## Linked List 절반 구하기

~~~python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class LinkedList:
    def halfOfLinkedList(self, head: ListNode) -> ListNode:
        fast = slow = head
        
        # slow is half of head
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next

        return slow
~~~

예시

head  
ListNode{val: 1, next: ListNode{val: 2, next: ListNode{val: 2, next: ListNode{val: 1, next: None}}}}

slow  
ListNode{val: 2, next: ListNode{val: 1, next: None}}

## Linked List 역순

~~~python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class LinkedList:
    def ReverseOfLinkedList(self, head: ListNode) -> ListNode:
        node = None
        
        while head:
            nxt = head.next
            head.next = node
            node = head
            head = nxt

        return node
~~~

예시

head  
ListNode{val: 2, next: ListNode{val: 1, next: None}}

node  
ListNode{val: 1, next: ListNode{val: 2, next: None}}

## Class Memory 공유

~~~python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
        dummy = cur = ListNode(0)
        print(id(dummy)) # 140375281581984
        print(id(cur))   # 140375281581984
        # 주소값 동일 따라서 dummy변수에 대해서 연산을 안해도 데이터가 들어가 있는 걸 볼 수 있음.

        while l1 and l2:
            if l1.val < l2.val:
                cur.next = ListNode(l1.val)
                # cur.next = l1
                l1 = l1.next
            else:
                cur.next = ListNode(l2.val)
                # cur.next = l2
                l2 = l2.next
            
            cur = cur.next
            
        # 남은거 정리
        cur.next = l1 or l2
        return dummy.next
~~~
