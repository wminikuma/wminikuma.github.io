---
title: ArrayList는 무엇인가? 
author: Jeon Jihoon
date: 2023-03-15 23:10:06:00 +0800
categories: [IT, Java]
tags: [java, arraylist]
render_with_liquid: false
---

이번 포스팅은 Java에서 많이 사용하는 자료구조 중에 하나인 ```ArrayList```에 대해 알아보려고 한다.


## **Java ArrayList API에는 뭐라고 쓰여 있을까?**
---
> **Resizable-array** implementation of the List interface. Implements all optional list operations, and permits all elements, including null. In addition to implementing the List interface, this class provides methods to manipulate the size of the array that is used internally to store the list. (This class is roughly equivalent to Vector, except that it is unsynchronized.) The size, isEmpty, get, set, iterator, and listIterator operations run in constant time. The add operation runs in amortized constant time, that is, adding n elements requires O(n) time. All of the other operations run in linear time (roughly speaking). The constant factor is low compared to that for the LinkedList implementation. Each ArrayList instance has a capacity. The capacity is the size of the array used to store the elements in the list. It is always at least as large as the list size. As elements are added to an ArrayList, its capacity grows automatically. The details of the growth policy are not specified beyond the fact that adding an element has constant amortized time cost. An application can increase the capacity of an ArrayList instance before adding a large number of elements using the ensureCapacity operation. This may reduce the amount of incremental reallocation. Note that this implementation is **not synchronized**. If multiple threads access an ArrayList instance concurrently, and at least one of the threads modifies the list structurally, it must be synchronized externally. (A structural modification is any operation that adds or deletes one or more elements, or explicitly resizes the backing array; merely setting the value of an element is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the list. If no such object exists, the list should be "wrapped" using the Collections.synchronizedList method. This is best done at creation time, to prevent accidental unsynchronized access to the list:
{: .prompt-tip }

```java
List list = Collections.synchronizedList(new ArrayList(...));
```
 
> The iterators returned by this class's iterator and listIterator methods are **fail-fast**: if the list is structurally modified at any time after the iterator is created, in any way except through the iterator's own remove or add methods, the iterator will throw a ConcurrentModificationException. Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future. Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw ConcurrentModificationException on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: the fail-fast behavior of iterators should be used only to detect bugs. 
{: .prompt-tip }

생각과는 다르게 많은 내용을 API 문서에서 이야기를 하고 있다.     
이 중에 **```Resizable-Array```**, **```Not Synchronized```**, **```Fail-Fast```** 키워드 중심으로 이야기를 해보려고 한다.


## **가변 크기 배열**
---
API 문서에 보면 **가변 크기 배열**이라는 설명이 나오는 데 몇가지 요약을 해 보면 아래와 같다.

* 배열의 사이즈를 조정할 수 있는 ```List``` 인터페이스의 구현체
* 배열의 Element를 저장하기 위한 Capacity가 존재하며 크기는 항상 리스트의 크기보다 크다.
* Element가 ```ArrayList```에 추가되면 Capacity가 자동으로 증가
* 증가하는 배열 크기는 분할상환시간과 관련이 있음

여기서 처음을 생각해 봐야 할 내용은 배열 사이즈가 자동으로 증가된다는 점이다.  

```java
private static final int DEFAULT_CAPACITY = 10;
```

JDK 1.8 기준으로 ```ArrayList``` 배열의 기본 사이즈는 10으로 세팅되어 있다. 만약 배열의 기본 사이즈보다 더 커지는 경우에는 어떻게 될까? 아래 코드 일부를 살펴보자.

```java
/**
* Increases the capacity to ensure that it can hold at least the
* number of elements specified by the minimum capacity argument.
*
*  @param minCapacity the desired minimum capacity
*/
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```    

위 ```grow``` 메서드를 보게 되면 해당 메서드가 자동 증가와 관련이 있음을 알 수 있다. 자동 증가 수치는  ```int newCapacity = oldCapacit+(oldCapacity >> 1)``` 이 부분을 보면 알 수 있게 되는데, 새로운 배열의 크기는 예전 배열의 크기를 오른쪽으로 1만큼 이동한 수를 더해서 만들게 된다.

예를 들어 기본 배열 크기보다 더 큰 배열이 필요하다고 가정해 보면, 새로운 배열은 15의 크기만큼 할당될 것이라는 걸 알 수 있다.

```java
int newCapacity = oldCapacit+(oldCapacity >> 1)
    15          = 10        + 5

(oldCapacity >> 1): 1010(10의 이진수) 오른쪽 1만큼 이동 0101(5)
```    

일반화를 해 보면 ```int newCapacity = oldCapacit+(oldCapacity / 2)``` 로 나타낼 수 있다. 참고로 자동 증가된 배열은 ```Arrays.copyOf``` 을 보게 되면 기본 배열을 복사한 새로운 배열로 생성된다.    

## **Multiple Threading**
---
API 문서를 참고해 보면 ```ArrayList```는 Syncronized 가 아니다. 만약 멀티스레드 환경에서 ```ArrayList``` 인스턴스를 동시에 접근하거나 이 중 하나의 스레드에서 리스트의 구조를 수정하는 경우에는 외부 동기화 과정이 필요하다. 여기서 말하는 리스트의 구조를 변경하는 거는 element 의 추가, 삭제, 내부 배열 크기 수정, 값의 변경을 의미하지 않는다.    
외부 동기화는 일반적으로 ```Object```를 동기화하는 방식으로 해결할 수 있다. ```List``` 를 ```Collections.synchronizedList```와 같은 메서드로 Wrapping 하는 방법도 있다.

## **Fail-Fast**
---
```ArrayList``` 클래스도 ```Collection``` 클래스의 특징을 모두 가지고 있다. ```Collection``` 객체는 저장된 객체들에 대한 순차적 접근을 허용하는데 순차적인 접근 시, ```Collection``` 객체의 구조적인 변경, 예를들어 객체의 추가 혹은 삭제와 같은 일이 발생하게 되면 접근 실패(```ConcurrentModificationException```) 예외가 발생하게 된다.

```java
public class FailFast {
    public static void main(String[] args) {
        ArrayList<Integer> lists = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        Iterator<Integer> iter = lists.iterator();
        while (iter.hasNext()) {
            Integer number=  iter.next();
            lists.add(50);  // collection 구조 변경
            System.out.println(number);
        }
   }
}
```

## **My ArrayList 만들기**
---
아래 몇가지 기능을 정의하여 직접 ```ArrayList```만들어 보려고 한다. 기능은 JDK 8 스펙을 참고 하였다.  

* Default Capacity는 10
* 생성자는 파라미터가 없는 경우, 파라미터가 1개 존재하는 경우만 고려
* 리스트의 추가 기능은 맨 뒤에 추가하는 기능과 특정 인덱스에 추가하는 기능만 고려
* 특정 인덱스의 값을 삭제하는 기능
* 배열의 크기가 꽉 찼을 경우 자동으로 증가하는 기능 고려
* 배열의 리스트가 꽉 찼는지 리스트 추가 전에 체크
* 특정 인덱스 값을 조회하는 기능


```java
import java.util.Arrays;

public class MyArrayList<E> {
    //Initial Capacity
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private Object[] elementData = {};
    private int index = 0;

    //constructor no parameter
    public MyArrayList() {
        this.elementData = new Object[DEFAULT_CAPACITY];
    }
    //constructor 1st parameter
    public MyArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("IllegalArgumentException: " + initialCapacity);
        }
    }
    //array size
    public int getSize() {
        return this.elementData.length;
    }
    //last add 
    public void add(E e) {
        int finalIndexPos = getValueIndex();
        ensureChecked(finalIndexPos);
        this.elementData[index] = e;
        index++;
    }
    // add by index
    public void add(int index, E e) {
        int finalIndexPos = getValueIndex();
        ensureChecked(finalIndexPos);
        rightSwiftArray(finalIndexPos, index, e);
    }
    //remove
    public void remove(int index) {
        int finalIndexPos = getValueIndex();
        leftSwiftArray(finalIndexPos, index);
    }
    //get
    @SuppressWarnings (value="unchecked")
    public E get(int index) {
        return (E) this.elementData[index];
    }

    private int getValueIndex() {
        int findValueIndex = 0;
        for (int i = 0; i < this.elementData.length; i++) {
            if (this.elementData[i] == null) {
                break;
            }
            findValueIndex++;
        }
        return findValueIndex;
    }
    //length checked + size up
    private boolean ensureChecked(int finalIndexPos) {
        if (index >= elementData.length || finalIndexPos >= this.elementData.length) {
            grow(); //Size up
            return true;
        } else if (index < 0) {
            throw new IndexOutOfBoundsException("IndexOutOfBoundsException: " + index);
        }
        return false;
    }
    //grow
    private void grow() {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity * 2;  // doubling
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //array right swift
    private void rightSwiftArray(int finalIndexPos, int index, E e) {
        for (int i = finalIndexPos-1; i >= index; i--) {
            this.elementData[i+1] = this.elementData[i] ;
        }
        this.elementData[index] = e;
    }
    //array left swift
    private void leftSwiftArray(int finalIndexPos, int index) {
        for (int i = index; i < finalIndexPos; i++) {
            this.elementData[i] = this.elementData[i+1];
        }
        this.elementData[finalIndexPos-1] = null;
    }
}
```    

## **References**
---
* https://daehwann.wordpress.com/2016/08/26/java-arraylist/    
* https://docs.oracle.com/javase/8/docs/api/ 
* 코딩 인터뷰 완전 분석