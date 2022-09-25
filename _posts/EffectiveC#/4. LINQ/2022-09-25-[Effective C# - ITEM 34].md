---
title: EFC# ITEM 34. 함수를 매개변수로 사용하여 결합도를 낮추라.
date: 2022-09-25 20:46:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

함수를 매개변수로 취한다는 것은 개발자가 더 이상 구상 타입(concreate type)을 작성할 필요가 없으며, 오히려 추상화된 정릐를 통해 종속성을 다루는 것을 의미한다.

## ✅ 상속
- 베이스 클래스를 작성하여 이를 상속하는 방식이다. 이 방식을 사용하면 최종 사용자는 우리가 개발한 컴포넌트를 매우 쉽게 사용할 수 있다.
- 이 경우 계약 사항은 매우 명확하며 특정 베이스 클래스를 상혹하여 새로운 클래스를 작성하고 추상(혹은 가상) 메서드를 구현해 주기만 하면 된다. 자주 사용되는 기능은 추상 베이스 클래스에 미리 구현해둘 수도 있다.
- **장점**
  - 사용자는 동일한 기능을 재구현해야하는 부담을 덜 수 있다.
  - 파생 클래스가 모든 추상 메서드를 구현하지 않으면 컴파일러가 빌드 과정에서 오류를 발생 시킨다.
- **단점**
  - 베이스 클래스에서 파생 클래스가 반드시 작성해야하는 작업을 강제하는 것은 가장 제약이 많은 방법이다.

## ✅ 인터페이스
- 인터페이스를 작성하고 이를 구현하도록 하는 방식이다
- **장점**
  - 베이스 클래스에 의존하는 방식보다는 조금 더 느슨한 결합을 만들 수 있다.
  - 사용자에게 클래스의 계층 구조를 강제하지 않을 수 있다.
- **단점**
  - 인터페이스만을 제공하기 때문에 클라이언트 코드에서 활용할 수 있는 재사용 기능 코드를 제공하기 어렵다.

## ✅ 함수를 매개변수로 취하는 델리게이트
- 2개의 시퀀스를 연결하는 Zip 메서드 예제이다.
```csharp
public static IEnumerable<string> Zip(IEnumerable<string> first, IEnumerable<string> second)
{
    using(var firstSequence = first.GetEnumerator())
    {
        using(var secondSequence = second.GetEnumerator())
        {
            while(firstSequence.MoveNext() && secondSequence.MoveNext())
                yield return $"{firstSequence.Current} {secondSequence.Current}";
        }
    }
}
```
- 이 코드를 조금 수정하여 출력 시퀀스를 만들기 위한 함수를 매개변수로 받도록 개선할 수 있다.
```csharp
public static IEnumerable<TResult> Zip<T1, T2, TResult>(
    IEnumerable<T1> first, 
    IEnumerable<T2> second,
    Func<T1, T2, TResult> zipper)
{
    using (var firstSequence = first.GetEnumerator())
    {
        using (var secondSequence = second.GetEnumerator())
        {
            while (firstSequence.MoveNext() && secondSequence.MoveNext())
                yield return zipper(firstSequence.Current, secondSequence.Current);
        }
    }
}
```
```csharp
var result = Zip(first, second, (one, two) => $"{one} {two}");
```
- 하지만 이렇게 결합도를 느슨하게 구성하려면 오류를 처리하기 위해서 추가적인 작업이 필요하다.
- 이벤트를 발생시킬 때 해당 이벤트가 null인지 확인해야하고, 델리게이트가 null 인경우 이를 위해 기본동작을 마련해야할지 등 여러 생각할 거리가 생긴다.
  
## ⭐정리⭐
- 예측할 수 있는 수준에서 델리게이트를 잘 활용하면 유연성을 매우 높일 수 있다.

