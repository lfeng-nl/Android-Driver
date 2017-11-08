# Linux 中链表的常用操作
- 必须包含头文件`<linux/list.h>`，该文件定义了一个简单的`list_head`类型的结构体；
	```c
	struct list_head {
		struct list_head *next, *prev;
	};
	```
- 实际代码的链表几乎都是结构体类型构成，每个结构体描述链表中的一项。需要使用链表只需要在结构体里嵌入一个list_head:
	```c
	struct my_struct{
		struct list_hand list;
		....
		void *my_data;
	}
	```
- 链表在使用前必须用`INIT_LIST_HEAD`来初始化；
	```c
	static inline void INIT_LIST_HEAD(struct list_head *list)
	{
		list->next = list;
		list->prev = list;
	}
	```
- 头文件`<linux/list.h>`中声明了list的相关操作
```c
/* 在链表头部添加新项，通常是链表头部，这样可以用来建立栈 */
void list_add(struct list_head *new, struct list_head *head);

/* Insert a new entry before the specified head.  This is useful for implementing queues. */
void list_add_tail(struct list_head *new, struct list_head *head);

/* deletes entry from list. 删除列表中的指定项 */
void list_del(struct list_head *entry);
/*  deletes entry from list and reinitialize it.删除列表中的指定项，并初始它（有可能插入到其他链表中） */
void list_del_init(struct list_head *entry);

/* tests whether a list is empty.如果链表为空，返回非零值 */
int list_empty(const struct list_head *head);

/**
 * iterate over a list，遍历链表
 * @pos:	the &struct list_head to use as a loop cursor.
 * @head:	the head for your list.
 */
list_for_each(pos, head);

/**
 * list_for_each_entry	-	iterate over list of given type
 * @pos:	the type * to use as a loop cursor.
 * @head:	the head for your list.
 * @member:	the name of the list_struct within the struct.
 */
list_for_each_entry(pos, head, member)				\
	for (pos = list_entry((head)->next, typeof(*pos), member);	\
	     &pos->member != (head); 	\
	     pos = list_entry(pos->member.next, typeof(*pos), member))
```