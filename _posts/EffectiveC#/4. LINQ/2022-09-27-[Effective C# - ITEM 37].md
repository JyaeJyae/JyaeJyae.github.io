---
title: EFC# ITEM 37. 쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다.
date: 2022-09-27 20:23:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

## ✅ 지연 평가
- 쿼리를 정의한다고 해서 결과 데이터나 시퀀스를 즉각적으로 얻어오는 것은 아니다.
- **지연평가** : 실제로 쿼리의 결과를 이용하여 순회를 수행해야만 결과가 생성된다.
- **즉시평가** : 마치 일반 변수를 사용하는 것처럼 즉각적으로 그 값을 얻어온다.
- 람다 표현식 인자는 즉각 수행되지 않으며 향후 필요한 시점에 수행될 코드 조각을 나타낸다.
```csharp
var sequence1 = Getnerate(10, () => DateTime.Now);
var sequence2 = from value in sequence1
                select value.ToUniversalTime();
```
- sequence1과 sequence2는 데이터를 공유하는 것이 아니라 내부적으로 함수를 합성하여 수행된다.
- sequence2를 순회할 때 sequence1이 이미 생성해둔 값을 순회하면서 개별 요소를 세계 공용 포맷으로 수정하는 것이 아니라, 순회 시접에 맞춰 sequence1에서 지정한 코드를 호출하여 그 결과를 얻은 후 연이어 세계 공용 포맷으로 그 내용을 변경한다.

## ✅ 무한 시퀀스
```csharp
static void Main(string[] args)
{
    var answers = from number in AllNumbers()
                    where number < 10
                    select number;
                    
    foreach(var num in small answers)
        Console.WriteLine(num);
}

static IEnumerable<int> AllNumbers()
{
    var number = 0;
    while(number < int.MaxValue)
    {
        yield return number++;
    }
}
```
- 하지만, 일부 쿼리 표현식의 경우 결과를 얻기 위해서 반드시 전체 시퀀스가 준비되어야 하는 경우도 있다.
- where, orderby, max, min은 전체 시퀀스를 요구한다.

## ✅ 전체 시퀀스를 사용하는 메서드를 사용할 때 주의사항
> 1. 시퀀스가 무한정 지속될 가능성이 있다면 이 같은 메서드를 사용할 수 없다.
> 2. 시퀀스가 무한이 아니더라도 **필터링은 다른 쿼리보다 먼저 수행**하는 것이 좋다. 개별 요소의 개수가 줄어 다음으로 수행할 쿼리의 성능을 개선할 수 있기 때문이다.
> 3. 쿼리가 즉각 실행되도록 하여 시퀀스를 실제 순환하기 이전에 지체 없이 데이터의 스냅샷을 얻고자 하는 경우에는 ToList()나 ToArray() 2개의 메서드를 사용하면 된다.

```csharp
// 정렬 후 필터링
var sortedProjuctsSlow = from p in products
                         orderby p.UnitsInStock descending
                         where p.UnitsInStock > 100
                         select p;
                         
// 필터링 후 정렬
var sortedProjuctsSlow = from p in products
                         where p.UnitsInStock > 100
                         orderby p.UnitsInStock descending
                         select p;
```

## ⭐정리⭐
- 대부분의 경우에 지연 평가를 사용하면 즉시 평가에 비해서 작업의 양도 줄고 유연성도 증가한다.
- 드문 경우이기 하지만 즉각적으로 쿼리를 수행하고 그 결과를 가져와야 하는 경우라면 ToList()나 ToArray()를 사용하면 된다.
- 하지만, 즉시 평가가 반드시 필요한 경우가 아니라면 대체로 지연평가를 사용하는 편이 낫다.

