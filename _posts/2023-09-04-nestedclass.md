---
title: "[Java] Nested Class"
date: 2023-09-04 14:53:00 +0900
categories: [IT, Java]
tags: [java, nestedclass]     # TAG names should always be lowercase
sitemap :
  changefreq : weekly
  priority : 1.0
---

이번 포스팅은 Nested Class에 대해 알려보려고 합니다. Oracle JDK 8 기준 Nested Classes 문서를 번역한 글입니다.


### **중첩 클래스는 무엇인가?**
---
자바에서는 클래스 내부에 클래스를 정의할 수 있다. 이것을 중첩 클래스 (```Nested Class```)라고 부른다.  아래는 중첩 클래스의 예시이다.    

```java    
class Outer {
  ...
  class NestedClass {
    ...
  }
}
```    

> **(전문용어)** 중첩 클래는 ```non-static``` 과 ```static``` 으로 구분할 수 있다. ```non-static``` 인 경우는 내부 클래스 (```Inner Class```) 라고 불리기도 한다. 반면에 ```static``` 같은 경우는 정적 중첩 클래스 (```Static Inner Class```) 라고 불린다.
{: .prompt-info }     


```java    
class OuterClass {
  ...
  class InnerClass {
    ...
  }

  static class StaticNestedClass {
    ...
  }
}
```       

중첩 클래스는 외부 클래스에 의해 둘러싸인 클래스이고 외부 클래스의 멤버로 동작한다. ```non-static``` 과 ```static``` 에 따라 조금은 다르게 동작한다. ```non-static``` 중첩 클래스 (내부 클래스) 경우에는 외부 클래스의 멤버에 접근이 가능하다. 설령 ```private``` 으로 선언된 멤버라고 해도 접근이 가능하다. 반면에 ```static``` 중첩 클래스는 외부 클래스의 멤버에 접근 할 수 없다. 참고로 외부 클래스의 멤버로 중첩 클래스는 ```private```, ```public```, ```protected``` 혹은  ```package private``` 으로 선언할 수 있다. (외부 클래스는 자체는 ```public``` 혹은 ```package private``` 으로 선언 가능)    


### **왜 중첩 클래스를 사용하는가?**  
---
중첩 클래스를 사용하는 이유는 아래와 같다.

* 한곳에서만 사용하고 있는 클래스를 논리적으로 그룹화 할 수 있다.     
  만약 클래스가 하나의 클래스에서만 사용되고 있다면, 클래스를 다른 클래스에 포함시켜 하나로 유지하는게 유용할 수 있다. ```Helper Class``` 같은 클래스를 중첩 클래스로 사용하면 패키지를 간소화 할 수 있다.
* 많은 캡슐화를 사용할 수 있다.
  최상위 클래스 A, B가 존재하고 B 클래스는 ```private``` 으로 선언된 A 클래스 멤버에 접근한다고 가정해 보자. (B 클래스 > A 클래스 ```Private```  멤버)
  클래스 A 내부에 B 클래스를 숨기게 되면 (내부 클래스 선언) B 클래스는 A 클래스의 ```Private```  멤버에 접근이 가능하게 된다. 또한 B 클래스 자체도 외부로 부터 숨길수 있다. 이것이 중첩 클래스를 활용한 캡슐화의 예시이다.
* 더 읽기 쉽고 유지보수가 좋은 코드를 만들 수 있다.
  최상위 클래스 내부에 중첩 클래스를 만들게 되면 연관성 있는 코드를 한곳에 두게 되어 코드의 사용성이 증가하게 된다.

### **내부 클래스**
---   

객체의 인스턴스 메서드 혹은 필드와 마찬가지로 내부 중첩 클래스는 감싸고 있는 외부 클래스의 인스턴스와 연결되어 외부 객체의 메서드 및 필드에 접근할 수 있지만 내부 클래스는 외부 클래스와 연결되어 있어 정적 멤버 (```static```) 를 정의할 수 없다. 참고로 JDK 16 + 버전에서는 허용되는 것으로 확인된다.     

(참고) JDK 16 이하 버전에서 내부 클래스 허용 멤버 접근자
* ```final``` (O)
* ```static + final``` (O)
* ```static``` (X)

```java
class OuterClass {
  ...
  class InnerClass {
    final int count = 100;        // OK
    static final int count = 100; // OK
    static int count = 100;       // Not OK
    ...
  }
}
```    

내부 클래스의 인스턴스 객체는 외부 클래의 인스턴스 내에 존재한다.     

```java
class OuterClass {
  ...
  class InnerClass {
    ...
  }
}
```    

```InnerClass``` 의 인스턴스는 ```OuterClass``` 의 인스턴스 내에서만 존재할 수 있으며 ```OuterClass``` 의 인스턴스의 메서드와 필드에 직접 접근할 수 있다. 내부 클래스를 인스턴스로 생성하기 위해서는 먼저 외부 클래스를 인스턴스로 생성해야 한다.    

```java
OuterClass outerObject = new OuterClass();
OuterClass.InnerClass innerObject = outerObject.new InnerClass();
```     

### **Static Nested Classes**
---

내부 클래스와 마찬가지로 정적 중첩 클래스도 외부 클래스외 연결되어 있다. 다만 정적 메서드와처럼 정적 중첩 클래스도 인스턴스 변수나 메서드를 직접 참조할 수 없고 객체 참조를 통해서만 접근이 가능하다.      

>**(Note)** 정적 중첩 클래스는 다른 최상위 클래스와 마찬가지로 외부 클래스 혹은 다른 클래스와 연결된다. 사실 정적 중첩 클래스는 동작으로 봤을 때 최상위 클래스라고 볼 수 있다. 
{: .prompt-info } 

다른 최상위 클래스와 마찬가지로 정적 중첩 클래스도 인스턴스를 아래와 같이 만들 수 있다.    

```java
StaticNestedClass staticNestedObject = new StaticNestedClass();
```    

### **Inner Class, Nested Static Class 예제**   
---

```OuterClass.java``` 를 아래와 같이 구성하자.      

```java
public class OuterClass {

    String outerField = "Outer field";
    static String staticOuterField = "Static outer field";

    class InnerClass {
        void accessMembers() {
            System.out.println(outerField);
            System.out.println(staticOuterField);
        }
    }

    static class StaticNestedClass {
        void accessMembers(OuterClass outer) {
            // Compiler error: Cannot make a static reference to the non-static
            //     field outerField
            // System.out.println(outerField);
            System.out.println(outer.outerField);
            System.out.println(staticOuterField);
        }
    }

    public static void main(String[] args) {
        System.out.println("Inner class:");
        System.out.println("------------");
        OuterClass outerObject = new OuterClass();
        OuterClass.InnerClass innerObject = outerObject.new InnerClass();
        innerObject.accessMembers();

        System.out.println("\nStatic nested class:");
        System.out.println("--------------------");
        StaticNestedClass staticNestedObject = new StaticNestedClass();        
        staticNestedObject.accessMembers(outerObject);
        
        System.out.println("\nTop-level class:");
        System.out.println("--------------------");
        TopLevelClass topLevelObject = new TopLevelClass();        
        topLevelObject.accessMembers(outerObject);                
    }
}
```     

```TopLevelClass.java``` 코드이다.    


```java
public class TopLevelClass {

    void accessMembers(OuterClass outer) {     
        // Compiler error: Cannot make a static reference to the non-static
        //     field OuterClass.outerField
        // System.out.println(OuterClass.outerField);
        System.out.println(outer.outerField);
        System.out.println(OuterClass.staticOuterField);
    }  
}
```      

정적 중첩 클래스 ```StaticNestedClass``` 는 둘러싸고 있는 외부 클래스인 ```OuterClass``` 의 인스턴스 멤버인 ```outField``` 에 직접 접근할 수 없다. (인스턴스 멤버에 접근하기 위해서는 인스턴스의 연결이 필요) 유사하게 최상위 클래스인 ```TopLevelClass``` 도 ```outField``` 에 직접 접근할 수 없다.

수정된 코드로 나온 결과물은 아래와 같다.

```text
Inner class:
------------
Outer field
Static outer field

Static nested class:
--------------------
Outer field
Static outer field

Top-level class:
--------------------
Outer field
Static outer field
```

### **Shadowing**    
---

만약 특정 범위 내부(내부 클래스 혹은 메서드)에서 선언된 타입들 (멤버 변수 혹은 매개변수 이름)이 외부 범위에 선언된 타입들과 이름이 같은 경우에 외부에 선언된 이름으로 대체된다. 이것을 Shadowing이라고 한다. 

```java
public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}
```     

```text
x = 23
this.x = 1
ShadowTest.this.x = 0
```      

**(예제 설명)**   
예제에서는 ```x``` 라는 이름의 3가지 변수를 정의한다. 각각 ```ShadowTest``` 클래스 멤버 변수, ```FirstLevel``` 내부 클래스의 멤버 변수, ```methodInFirstLevel``` 의 매개변수로 선언된 변수 입니다. 내부 클래스의 메서드인 ```methodInFirstLevel``` 의 매개변수로 정의된 변수 ```x``` 는 내부 클래스의 변수를 Shadowing 한다. 내부 클래스인 ```FirstLevel``` 의 멤버 변수인 ```x``` 를 참조하려면 ```this``` 키워드가 필요합니다.     

(1) ```this``` 키워드로 내부 클래스인 ```FirstLevel``` 의 멤버 변수를 접근
```java
System.out.println("this.x = " + this.x);
```    

(2) 상위 레벨 클래스의 멤버 변수를 참조해야 하는 경우 (해당 변수가 속한 클래스 명 작성)    

```java
System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
```      

### **Serialization**   
--- 

로컬 및 익명 클래스를 포함한 내부 클래스의 직렬화는 권장하지 않는다. 


### **References**    
---
[Oracle JDK 8 Reference Nested Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)
