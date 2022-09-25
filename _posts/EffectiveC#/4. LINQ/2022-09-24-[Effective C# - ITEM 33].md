---
title: EFC# ITEM 33. 필요한 시점에 필요한 요소를 생성하라.
date: 2022-09-24 22:09:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

이터레이터 메서드를 구현할 때 통상 새로운 출력 시퀀스를 생성하기 위해서 yield return을 주로 사용하게 되는데, 이 과정에서 입력 시퀀스를 활용하는 대신 새로운 요소를 생성하는 `팩토리 메서드`를 사용할 수도 있다.

작업을 수행하기 전에 필요한 요소를 모두 생성해서 컬렉션에 저장해두는 대신 필요할 때마다 개발 요소를 생성하는 식이다.

## ✅ 일반 메서드
```csharp
static IList<int> CreateSequence(int numberOfElements, int startAt, int stepBy)
{
    var collection = new List<int>(numberOfElements);
    for(int i = 0; i < numberOfElements; i++)
        collection.Add(startAt + i * stepBy);
        
    return collection;
}
```
- 시퀀스 내의 요소들을 모두 생성할 때까지 다음 단계를 진행할 수 없기 때문에 전체 파이프라인의 병목 구간이 된다.

## ✅ 이터레이터 메서드
```csharp
public static IEnumerable<int> CreateSequence(int numberOfElements, int startAt, int stepBy)
{
    for (int i = 0; i < numberOfElements; i++)
        yield return startAt + i * stepBy;
}
```
- 시퀀스 내의 개별 요소가 요청 시마다 하나씩 생성된다.

```csharp
var data = new BindingList<int>(CreateSequence(100, 0, 5).ToList());
```
- ToList()는 CreateSequence()가 생성한 시퀀스의 모든 요소들을 담고있는 List 객체를 생성하고 BindidngList<int> 객체는 이 List 객체를 참조하고 있는 구조가 된다.

## ✅ 이터레이터 메서드 중단
```csharp
var sequence = CreateSequence(100, 0, 5).TakeWhile((n) => n < 50);
```
- 시퀀스에 대한 순회 과정을 쉽게 중단할 수 있다.
  
## ⭐정리⭐
- 시퀀스 전체를 소비하지 않고 일부만 필요한 경우임에도 전체 요소를 미리 생성해두는 것은 좋은 방법이 아니다.
- 어떤 경우라도 필요한 시점에 맞추어 필요한 요소를 생성하도록 코딩하는 것이 가장 효과적이다.

