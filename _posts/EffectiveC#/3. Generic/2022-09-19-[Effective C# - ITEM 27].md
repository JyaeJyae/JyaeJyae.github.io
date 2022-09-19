---
title: EFC# ITEM 27. 인터페이스는 간략히 정의하고 기능의 확장은 확장 메서드를 사용하라.
date: 2022-09-19 18:07:55 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch3.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

확장 메서드를 이용하면 인터페이스에 새로운 동작을 추가할 수 있다. `인터페이스에는 가능한 최소한의 기능만을 정의`하고, 확장 메서드를 세트로 함께 구현하면 손쉽게 기능을 확장할 수 있다.

### System.Linq.Enumerable 클래스
IEnumerable<T>는 기본적인 구현만 가지고 있으며, Where, OrderBy, ThenBy, GroupInfo 등은 확장 메서드로 구현되어있다.
- 장점 : 이미 구현하고 있는 클래스를 수정할 필요가 없다.

## ✅ 인터페이스
```csharp
public interface IFoo
{
    int Marker { get; set; }
}

public static class FooExtensions
{
    public static void NextMarker(this IFoo thing) =>
        thing.Marker += 1;
}

MyType t = new MyType();
t.NextMarker();
```
- 어디서든 확장 메서드 사용할 수 있다.(static)

## ✅ 확장 메서드와 내부 메서드의 충돌
```csharp
public class MyType : IFoo
{
    public int Marker{ get; set; }
    public void NextMarker() => Marker += 5;
}
```
- Marker 값은 5가 된다.
- 의미적으로 완전히 동일한 작업이 수행되도록 작성해야한다.

## ⭐정리⭐
- 여러 클래스에서 반드시 구현해야하는 인터페이스를 정의하는 경우 인터페이스 내에 정의하는 멤버의 수를 최소한으로 해야한다.
- 사용자의 편의를 위해서 추가적으로 제공하려는 메서드는 확장 메서드 형태로 구현하는 것이 좋다.
