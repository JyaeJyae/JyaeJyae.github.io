---
title: EFC# ITEM 28. 확장 메서드를 이용하여 구체화된 제네릭 타입을 개선하라.
date: 2022-09-19 18:18:55 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch3.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

프로그램을 개발하다 보면 List<int>나 Dictionary<Custom>와 같이 제네릭 컬렉션에 타입 매개변수를 지정하여 사용하게 될 것이다. 기존에 사용 중인 컬렉션 타입에 영향을 주지 않으면서 새로운 기능을 추가하고 싶다면 구체화된 컬렉션 타입에 대해 확장 메서드를 작성하면 된다.

## ✅ IEnumerable에 대한 확장 메서드
```csharp
public static class Enumerable
{
  public static int Average(this IEnumerable<int> sequence);
  public static int Max(this IEnumerable<int> sequence);
  public static int Min(this IEnumerable<int> sequence);
  public static int Sum(this IEnumerable<int> sequence);
}
```
- 위 패턴은 타입 매개변수로 특정 타입이 주어질 때, 해당 타입에 대해 가장 효과적으로 동작하도록 코드 분리해서 구현하는 방법이다.

## ✅ 확장 메서드 vs 파생 클래스의 메서드
### 파생 클래스 메서드
```csharp
public class CustomerList : List<Customer>
{
    public void SendEmailCoupons(Coupon specialOffer)
    {
        //...
    }
}
```
### 확장 메서드
```csharp
public static void SendEmailCoupons(this IEnumerable<Customer> customers, Coupon specialOffer)
{
    //...
}
```

- 확장 메서드는 IEnumerable<Customer>를 기반으로 작성되었지만, 상속으로 파생된 클래스에 메서드를 추가한 방식은 List<Customer>를 기반으로 한다.
- 결국 더 이상 이터레이터 메서드를 사용할 수 없다.

## ⭐정리⭐
- 기능을 추가할 때 구체화된 제네릭 필드를 상속하여 메서드를 추가하기 보다는 확장메서드를 구현하는 것을 고려하자.