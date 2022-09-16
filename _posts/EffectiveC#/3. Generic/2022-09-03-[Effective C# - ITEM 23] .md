---
title: EFC# ITEM 23. 타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 활용하라.
date: 2022-09-03 22:31:55 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch3.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

C#에서 제약 조건을 설정하는 방법에는 한계가 많다.
- 임의의 정적 메서드(연산자를 포함하여)를 반드시 구현해야 하는 것
- 매개변수를 취하는 여타의 생성자를 반드시 구현해야 하는 것

> **C#에서 제약 조건을 설정하는 방법**
> - class 타입이나 struct 타입으로 형태 제한
> - 매개변수가 없는 생성자를 가져야하는 조건으로 제한

## ✅ 인터페이스를 이용한 메서드 제약
- 어떤 제네릭 클래스에 대해 타입 매개변수 T가 반드시 Add()메서드를 가져야 한다는 제약 조건을 설정하고 싶다고 가정한다.
  1. Add() 메서드를 정의하는 IAdd<T> 인터페이스 생성
  2. 이 인터페이스를 구현한 클래스 생성, IAdd<T>가 정의한 Add() 메서드를 구현
  3. 제네릭 클래스의 정의를 이용하여 닫힌 제네릭 클래스 생성

## ✅ 델리게이트를 이용한 메서드 제약
- 제약 조건으로 델리게이트를 작성하면 추가적인 작업 없이 해결 가능하다.
- 특정 제네릭 클래스가 T타입의 두 객체를 더하는 메서드를 필요로 한다고 가정하자.
- System.Func<T1, T2, TOutput> 델리게이트를 사용하면 필요한 원형의 델리게이트를 얻을 수 있다.
```csharp
// Add 메서드 구현
public static class Example
{
    public static T Add<T>(T left, T right, Func<T, T, T> AddFunc) =>
        AddFunc(left, right);
}
```
```csharp
// Example.Add의 사용
int a= 6;
int b = 7;
int sum = Example.Add(a, b, (x, y) => x + y);
```

## ⭐정리⭐
- `설계의 특성을 드러내는 가장 좋은 방법은 class나 인터페이스로 제약 조건을 설정`하는 것이다.
- 하지만, 제네릭 클래스나 제네릭 메서드 하나를 사용하기 위해서 인터페이스를 구현한 타입을 매번 새롭게 만드는 것은 매우 번거롭다.
- `델리게이트`를 이용하면 제네릭 타입을 더 쉽게 작성할 수 있고 사용하기 편하다.
