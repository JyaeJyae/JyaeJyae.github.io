---
title: EFC# ITEM 31. 시퀀스에 사용할 수 있는 조합 가능한 API를 작성하라.
date: 2022-09-22 20:51:32 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

foreach, for, while 등을 주로 사용해 컬렉션을 반환하는 반복문을 작성하지만, `효율성 문제`가 있다. 일반적으로 반복 구문을 포함하는 메서드를 작성하는 경우에 매개변수로 컬렉션을 받아와서 컬렉션에 포함된 요소들을 살펴보거나, 내용을 수정하거나, 혹은 그 중 일부만 필터링해서 또 다른 컬렉션에 그 결과를 저장한 후 반환하는 식으로 코드를 작성하게 된다.

하지만 대부분의 경우 이러한 작업들은 단일 작업이 아니라 여러 작업을 거쳐야 하며, 각 단계를 거칠 때마다 중간 결과를 저장하기 위해서 추가적으로 메모리 공간이 필요하기도 하며, 매번 전체 컬렉션을 순회하기 때문에 큰 비용이 발생하게 된다.

대안으로, 개별 요소에 대해 수행해야 하는 모든 작업을 분리된 메서드로 작성한 후 루프내에서 이 메서드를 호출하는 방법이 있다. 메서드의 재사용성이 낮아지는 문제가 있다.

이럴 때 `이터레이터 메서드`를 만들 수 있다.

## ✅ 이터레이터 메서드
- 단일의 시퀀스 IEnumerable<T>로 표현되는 입력을 취하고, 결과로도 단일의 시퀀스 IEnumerable<T>를 반환하는 메서드를 말한다.
- 이 메서드를 작성할 때 `yield return`을 사용하면 메서드 내에서 시퀀스 내의 개별 요소를 저장하기 위해 별도의 저장소를 마련할 필요가 없다.
- 정확히 값이 필요한 시점에 입력 시퀀스 상에서 다음 요소를 가져오고, 출력 결과가 반드시 필요한 시점에 출력 시퀀스로 결과를 내보내기 때문이다.

### Foreach문
```csharp
public static void Unique(IEnumerable<int> nums)
{
    var uniqueVals = new HashSet<int>();

    foreach (var num in nums)
    {
        if(!uniqueVals.Contains(num))
        {
            uniqueVals.Add(num);
            Console.WriteLine(num);
        }
    }
}
```
### 이터레이터 메서드
```csharp
public static IEnumerable<int> Unique2(IEnumerable<int> nums)
{
    var uniqueVals = new HashSet<int>();

    foreach (var num in nums)
    {
        if (!uniqueVals.Contains(num))
        {
            uniqueVals.Add(num);
            yield return num;
        }
    }
}
```
- yield return문은 값을 반환한 후 내부적으로 사용하는 이터레이터의 현재 위치와 상태 정보를 저장한다.
- 순회 과정은 내부적으로 입력 시퀀스의 현재 위치를 계속 갱신해가면서 출력 시퀀스에 차례차례 그 결과를 반환하는 과정이다.

## ✅ 이터레이터 메서드 조합
```csharp
public static IEnumerable<T> Unique<T>(IEnumerable<T> nums)
{
    var uniqueVals = new HashSet<T>();

    foreach (var num in nums)
    {
        if (!uniqueVals.Contains(num))
        {
            uniqueVals.Add(num);
            yield return num;
        }
    }
}

public static IEnumerable<int> Square(IEnumerable<int> nums)
{
    foreach (var num in nums)
        yield return num * num;
}
```
- 제곱을 만드는 이터레이터 메서드를 만들고, 기존 출력 메서드와 결합하는 예제이다.

## ✅ 복수의 입력시퀀스를 사용한 이터레이터 메서드
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
- 두개의 시퀀스를 연결 시켜 이를 결합한 후 단일 string 출력 시퀀스를 내보내는 예제이다.

## ⭐정리⭐
- 이터레이터 메서드를 하나의 입력 시퀀스와 하나의 출력 시퀀스를 갖도록 작성하면 조합하기가 쉬워진다.

