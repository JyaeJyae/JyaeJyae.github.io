---
title: EFC# ITEM 30. 루프보다 쿼리 구문이 낫다.
date: 2022-09-21 21:25:32 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

`쿼리 구문을 사용하는 것이 반복문을 사용하는 것보다 더 나은 경우가 꽤 있다.`

## ✅ 쿼리 구문의 장점
1. 프로그램의 논리를 명령형 방식에서 선언적 방식으로 전환할 수 있다.
2. 질의의 내용을 구성할 수 있을 뿐 아니라 개별 항목에 대해 수행하려는 작업의 수행 시기를 연기할 수 있다.
3. 사용자의 의도를 더 명확하게 드러낼 수 있다.

### (0, 0)에서 떨어진 거리 순으로 정렬하여 객체를 반환
- 루프
```csharp
private static IEnumerable<Tuple<int, int>> ProduceIndices()
{
   var storage = new List<Tuple<int, int>>();

    for (int x = 0; x < 100; x++)
    {
    	for(int y = 0; y < 100; y++)
        {
        	if (x + y < 100)
            	storage.Add(Tuple.Create(x, y));
        }
    }

    storage.Sort((point1, point2) =>
    	(point2.Item1 * point2.Item1 + point2.Item2 * point2.Item2).CompareTo(
         point1.Item1 * point1.Item1 + point1.Item2 * point1.Item2));
    return storage;
}
```
- 쿼리
```csharp
private static IEnumerable<Tuple<int, int>> QueryIndices()
{
	return result = from x in Enumerable.Range(0, 100)
                  	from y in Enumerable.Range(0, 100)
                  	where x + y < 100
                  	orderby (x * x + y * y) descending
                  	select Tuple.Create(x, y);
}
```

## ✅ 이터레이터 메서드를 사용하는 이유
- 시퀀스의 모든 원소를 한번에 만드는 것이 아니라 최종 시퀀스의 원소는 여러 과정을 거쳐 만드는 것을 가정할 때 이터레이터 메서드를 사용하는 것이 유리하다.
- 만약 즉시 평가된 컬렉션을 원한다면, ToList()나 ToArray()와 같은 확장 메서드를 이용하여 IEnumerable<T> 타입을 이용하는 시퀀스로부터 모든 항목이 담긴 컬렉션을 손쉽게 생성할 수 있다.



## ⭐정리⭐
- 반복 구문을 사용해야 한다면 이를 쿼리 구문으로 작성할 수 있는지 생각해보고 쿼리 구문으로도 어렵다면 메서드 호출 구문까지 고려해보자. 
- 거의 대부분의 경우 반복 구문을 사용하는 것보다 훨씬 깔끔하게 코드를 작성할 수 있을 것이다.

