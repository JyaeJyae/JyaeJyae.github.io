---
title: EFC# ITEM 32. Action, Predicate, Function과 순회 방식을 분리하라.
date: 2022-09-23 22:07:32 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

이터레이터 메서드는 크게 두 종류가 있다.
1. 시퀀스 내에서 개별 항목을 이용하여 작업을 수행하는 메서드
2. 시퀀스의 순회 방식을 변경하는 메서드


일반적으로 개발자들이 여러 가지 작업을 하나의 메서드에 집어넣어 개발하는 이유는 메서드 중간 어딘가를 커스텀화하기 어렵기 때문이다. `델리게이트`를 사용하면 메서드를 호출할 수 있는 무엇인가를 전달하거나 함수 자체를 전달할 수 있다.

## ✅ Predicate, Action, Func
```csharp
namespace System
{
    public delegate bool Predicate<T>(T obj);
    public delegate void Action<T>(T obj);
    public delegate TResult Func<T, TResult>(T arg);
}
```
> **💡익명 델리게이트 패턴**
>
> - **Function** : 반환 값이 있는 함수를 전달하는 델리게이트
> - **Predicate** : 시퀀스 내의 항목이 조건에 부합하는지를 Boolean으로 반환하는 함수
> - **Action** :  컬렉션 개별 요소에 대해 수행할 작업을 전달하는 델리게이트
{: .prompt-info }

## ✅ 필터 메서드
```csharp
public static IEnumerable<T> Where<T>(IEnumerable<T> sequence, Predicate<T> filterFunc)
{
    if(sequence == null)
        throw new ArgumentNullException(nameof(sequence), "sequence must not be null");
        
    if(filterFunc == null)
        throw new ArgumentNullException("Oreducate must not be null");
    
    foreach(T item in sequence)
        if(filterFunc(item)
            yield return item;
}
```
- 필터 메서드는 Predicate 이용하여 테스트를 수행한다.
- Predicate는 어떤 객체를 통과시키고 통과시키지 않을지를 결정하는 필터링 조건을 정의하기 위해 사용된다.

## ✅ 변환 메서드
```csharp
public static IEnumerable<Tout> Select<T in, T out>(IEnumerable<T in> sequence, Func<T in, T out> method)
{
    // 위와 동일하게 null체크는 해야함
    
    foreach(Tin element in sequence)
        yield return method(element);
}
```
- 변환 메서드는 Func을 통해 작성할 수 있다.
  
## ⭐정리⭐
- Fuction을 이용하면 시퀀스의 개별 요소에 대해 적용할 다양한 작업을 구현할 수 있다.
- Action을 이용하면 시퀀스의 일부 요소를 가져와서 수행할 작업을 손쉽게 구현할 수 있다.

