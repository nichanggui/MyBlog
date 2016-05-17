---
title: 设计模式之Iterator
tags: Iterator
---


**Iteraotr设计模**是在集合中使 用 的非常多的一种模式, Iterator 为我们提供了统一的遍历集合的方法,不管该种集合的底层是用数组还是链表实现.现在使用Iterator这种设计模式设计自己的集合类型,为了大家方便理解,现将我自定义的集合名和J D K中提供的集合名保持一致。

**接下来写一个简单的Iterator设计模式的例子**
- 1、定义Iterator接口(注意是自定义的，而非jdk自带的)
``` java
public interface Iterator{
	//判断是否还有下一个元素
	boolean hasNext();
	//取得下一个元素
	Object next();
	//获得迭代器
	Iterator iterator();
}
```
<!--more-->
- 2、定义Collection接口，该接口继承Iteraotr接口
``` java
public interface Collection extends Iterator{
	//增加元素
	public void add(Object obj);
	//获得列表大小
	public int size();
}
```
- 3、定义ArrayList类，该类实现了Collection接口
``` java
public class ArrayList implements Collection{
	private Object[]objects=new Object[10];
	private int length=-1;
	private int current=-1;
	//添加元素
	public void add(Object obj){
		if(length<objects.length-1){
			objects[++length]=obj;
		}else{
			Object[]newobjects=new Object[objects.length+10];
			System.arraycopy(objects, 0, newobjects, 0, objects.length);
			objects=newobjects;
			this.add(obj);
		}
	}
	//获得指定位置的元素
	public Object get(int index){
		if(index<=length && index>=0){
			return objects[index];
	   }else{
			 throw new  ArrayIndexOutOfBoundsException();
		}
	}
	//获得集合的大小
	public int size(){
		return length+1;
	}
	//判断是否有下一个元素
	@Override
	public boolean hasNext(){
		// TODO Auto-generated method stub
		if(current<length)
			return true;
		return false;
	}
	@Override
	public Object next(){
		// TODO Auto-generated method stub
		return objects[++current];
	}
	//返回迭代器
	public Iterator iterator(){
		return this;
	}
}
```

- 4、定义LinkedList类，该类实现了Collection接口，并定义了一个节点类Node
``` java
public class Node{
	private Object object;
	private Node next;
	public Object getObject(){
		return object;
	}
	public void setObject(Object object){
		this.object = object;
	}
	public Node getNext(){
		return next;
	}
	public void setNext(Node next){
		this.next = next;
	}
	public Node(Object obj,Node node){
		this.object=obj;
		this.next=node;
	}
}
```
``` java
public class LinkedList implements Collection{
	private Node head=null;
	private Node tail=null;
	private Node now;
	private int current;
	private int length=0;
	public void add(Object obj){
		Node node=new Node(obj,null);
		if(head==null){
			head=node;
			tail=head;
			now=head;
		}else{
			tail.setNext(node);
			tail=node;
		}
		length++;
	}
	public int size(){
		return length;
	}
	@Override
	public boolean hasNext(){
		// TODO Auto-generated method stub
		if(current<length){
			return true;
		}else{
			return false;
		}
	}
	@Override
	public Object next(){
		// TODO Auto-generated method stub
		Object obj=now.getObject();
		now=now.getNext();
		current++;
		return obj;
	}
	public Iterator iterator(){
		return this;
	}
}
```
- 5、测试
``` java
public class Test{
	public static void main(String[] args){
		//Collection col=new ArrayList();
		Collection col=new LinkedList();
		for(int i=0;i<10;i++){
			col.add(i);
		}
		Iterator iter=col.iterator();
		while(iter.hasNext()){
			System.out.println(iter.next());
		}
	}
}
```
## 总结
通过使用Iterator,使得我们遍历任何集合的代码都一样，从而提高了代码的重用性和简易性。



