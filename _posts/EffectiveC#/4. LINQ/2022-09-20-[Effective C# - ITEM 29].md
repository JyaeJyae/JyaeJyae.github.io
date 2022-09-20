---
title: EFC# ITEM 29. 컬렉션을 반환하기보다 이터레이터를 반환하는 것이 낫다.
date: 2022-09-20 20:43:32 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

메서드를 작성하다 보면 단일의 객체를 반환하기보다 일련의 시퀀스를 반환해야 하는 경우가 종종 있다. `시퀀스를 반환하는 메서드를 작성해야 한다면 컬렉션을 반환하기보다는 이터레이터를 반환하는 것이 좋다.`

## ✅ Iterator Method
- 이터레이터 메서드(iterator method) 란 호출자가 요청한 시퀀스를 생성하기 위해 **yield return** 문을 사용하는 메서드를 말한다.
```csharp
public static IEnumerable<char> GenerateAlphabet()
{
    var letter = 'a';
    while(letter <= 'z')
    {
    	yield return letter;
    	++letter;
    }
}
```
- 이터레이터 메서드가 호출되면 컴파일러가 생성한 객체가 인스터스화된다. 
- 이후 시퀀스 내에 포함된 항목을 요청하면 비로소 시퀀스가 생성된다.

```csharp
var allNumbers = Enumerable.Range(0, int.MaxValue);
```
- 이 메서드를 호출하면 숫자 시퀀스를 만들어내는 객체를 생성한다.
- 호출 측에는 이터레이터 메서드의 결과값을 추가적인 컬렉션에 저장하지 않는 이상 방대한 결과치를 저장하기 위한 공간이 필요하지 않다.

> 💡필요할 때 생성
> - 이터레이터 메서드는 시퀀스를 생성하는 방법을 알고 있는 객체를 생성한다. 
> - 그리고 이 객체는 실제 시퀀스에 대한 접근이 이루어지는 경우에만 사용된다.

## ✅ 이터레이터 메서드를 사용하는 이유
- 시퀀스의 모든 원소를 한번에 만드는 것이 아니라 최종 시퀀스의 원소는 여러 과정을 거쳐 만드는 것을 가정할 때 이터레이터 메서드를 사용하는 것이 유리하다.
- 만약 즉시 평가된 컬렉션을 원한다면, ToList()나 ToArray()와 같은 확장 메서드를 이용하여 IEnumerable<T> 타입을 이용하는 시퀀스로부터 모든 항목이 담긴 컬렉션을 손쉽게 생성할 수 있다.



## ⭐정리⭐
- 지연평가로 인해 파라미터의 유효성 확인이 늦어질 경우 다른 메소드에 유효성 확인 로직을 넣고, 통과하면 이터레이터를 리턴으로 호출하는 방식을 쓸 수 있다.

