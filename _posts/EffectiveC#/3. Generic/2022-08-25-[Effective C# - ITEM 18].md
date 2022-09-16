---
title: EFC# ITEM 18. 반드시 필요한 제약 조건만 설정하라.
date: 2022-08-25 20:36:32 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch3.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: EFC# ITEM 18. 반드시 필요한 제약 조건만 설정하라.
---

`타입 매개변수에 대한 제약 조건(constraint)`은 클래스가 작업을 올바르게 수행하기 위해서 타입 매개변수로 전달할 수 있는 타입의 유형을 제한하는 방법이다.

개발자는 올바르게 작업을 수행하기 위한 최소한의 제약 조건만을 설정해야 한다.

👉 제약 조건을 설정하지 않으면? 
- 런타임에 더 많은 검사 수행해야한다. 더 자주 형변환을 해야하고, 리플렉션을 사용해야할 가능성이 커지고, 잘못된 타입으로 인해 런타임 오류가 발생할 가능성 또한 높아진다.

👉 불필요한 제약 조건을 설정하면?
- 이 클래스를 사용하기 위해서 과도하게 추가 작업을 애야 한다.

## ✅ 제약 조건의 이점
- 제약 조건을 설정하는 대신 형변환이나 런타임에 테스트를 수행하도록 작성한 예제.
  ```csharp
  public static bool AreEqual<T>(T left, T right)
  {
      if (left == null)
          return right == null;
  
      if(left is IComparable<T>)
      {
          IComparable<T> lval = left as   IComparable<T>;
          if (right is IComparable<T>)
              return lval.CompareTo(right) == 0;
  
          ......
      }
  }
  ```
  - 런타임에 Icomparavle<T> 인터페이스로 형변환이 가능한지 확인한 후 이 인터페이스가 정의하고 있는 메서드 사용한다.
- 제약 조건을 설정하여 동일한 메서드를 간단하게 구현한 예제.
  ```csharp
  public static bool AreEqual2<T>(T left, T right) where T : IComparable<T>
  {
      return left.CompareTo(right) == 0;
  }
  ```
  - `런타임에 발생할 가능성이 있는 오류를 컴파일타임에 확인할 수 있다.`
- 첫번째 예제에서는 코드를 통해 런타임 오류가 발생하지 않도록 했지만, 두번째 예제에서는 컴파일러를 통해 런타임 오류가 발생하지 않도록 했다.

> 💡TIP!
> - 제네릭 타입을 작성할 때 필요한 제약 조건이 있다면 반드시 이를 지정하자.
> - 제약 조건이 없다면 타입의 오용 가능성이 높아지고 사용자의 잘못된 예측으로 런타임 예외나 오류가 발생할 가능성이 높아지게 된다.

## ✅ 제약 조건의 주의점
- 때로는 제약 조건을 설정하여 해당 클래스의 사용을 어렵게 만드느니 런타임에 특정 인터페이스를 구현하고 있는지 혹은 특정 베이스 클래스를 상속한 타입인지를 확인한 후 사용하는 것이 좋은 경우도 있다.
- new대신 default()를 사용하면 new() 제약 사항이 필요 없을 수도 있다.
  - `default() 연산자는 특정 타입의 기본값을 가져오는데, 값 타입에 대해서는 0을, 참조 타입에 대해서는 null을 가져온다.`
- new(), struct, class를 제약조건으로 설정하는 경우 주의해야한다.



## ⭐정리⭐
- 타입 매개변수를 지정하여 런타임에 발생할 오류를 줄이고 코드도 단축시킬 수 있다.
- 하지만, 과도하게 제약 조건을 사용하는 것은 오히려 독이 될 수 있으니 주의하자.

