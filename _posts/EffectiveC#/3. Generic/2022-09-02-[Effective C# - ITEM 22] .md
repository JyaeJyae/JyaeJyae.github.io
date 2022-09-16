---
title: EFC# ITEM 22. 공변성과 반공변성을 지원하라.
date: 2022-09-02 23:05:55 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch3.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

타입의 가변성(variance), 즉 `공변(convariance)`과 `반공변(contravariance)`은 특정 타입의 객체를 다른 타입 객체로 변환할 수 있는 성격을 말한다. 

## ✅ 공변과 반공변
 - 타입 매개변수로 주어지는 타입이 상호 호환 가능할 경우 이를 이용하는 제네릭 타입도 호환 가능함을 추론하는 기능이다.
 - IEnumerable<Object> 를 매개변수로 취하는 메서드는 IEnumerable<MyType> 객체도 받아들일 수 있어야 한다.
 - IEnumerable<MyType>객체를 반환하는 메서드는 이 반환 객체를 IEnumerable<Object> 객체에 할당할 수 있어야 한다.

### 공변(out)
 - X -> Y일 때, C<T>를 C<X> -> C<Y>인 경우 공변이다.
 - out 데코레이터를 사용하여 정의할 수 있다.

### 반공변(in)
 - Y -> X일 때, C<T>를 C<X> -> C<Y>인 경우 반공변이다.
 - in 데코레이터를 사용하여 정의할 수 있다.

```csharp
abctract public class CelestialBody : IComparable<CelestialBody>
{
	public double Mass { get; set; }
	public string Name { get; set; }
}

public class Planet : CelestialBody
{
}

public class Moon : CelestialBody
{
}

public class Asteroid : CelestialBody
{
}
```

```csharp
public interface IEnumerable<out T> : IEnumerable
{
	new IEnumerator<T> GetEnumerator();
}

public interface IEnumerator<out T> : IDisposable, IEnumerator
{
	new T Current { get; }
}
```

```csharp
public static void CoVariantArray(CelestialBody[] baseItems)
{
    foreach (var thing in baseItems)
    {
    	Console.WriteLine($"{this.Name} has a mass of {thing.Mass} Kg");
    }
    //반환되는 타입이 정확하므로 에러를 발생하지 않는다.
}
```

```csharp
public static void UnsafeVariantArray(CelestialBody[] baseItems)
{
    baseItems[0] = new Asteroid
    { Name = "Hygiea", Mass = 8.85e19 };
        
    //자식 클래스여도 타입이 다르기 때문에 예외를 일으킨다.
    
    CelestialBody[] spaceJunk = new Asteroid[5];
    spaceJunk[0] = new Planet();
    //마찬가지로 타입이 다르기 때문에 예외가 발생
}
```

## ✅ Out 키워드
- 제네릭 타입은 모두 불변이었으며 타입 매개변수가 다른 경우 대체가 불가능 했지만, C# 4.0이후부터는 공변과 반공변을 통해서 제네릭 타입의 대체 가능성을 지정할 수 있도록 개선되었다.

```csharp
public static void CovariantGeneric(IEnumerable<CelestialBody> baseItems)
{
    foreach (var item in baseItems)
    {
        Console.WriteLine($"{item.Name} {item.Mass}");
    }
}
```
- CovariantGeneric 메서드는 매개변수로 IEnumerable<CelestialBody>를 취하지만 List<Planet> 타입의 객체를 인자로 전달 할 수 있다.
- 이것이 가능한 이유는 IEnumerable<T> 인터페이스가 정의될 때 T를 `out키워드`로 선언했기 때문이다. 
```csharp
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}
```
- 이는 타입 매개변수 T를 출력 위치에서만 사용하겠다고 컴파일러에게 알려주는 것이다.
(반환 값, 속성의 get 접근자, 델리게이트 일부 위치)
- 내용을 조회는 하지만 수정을 하지 않음으로 공변을 지원한다.
- IList 컬렉션을 매개변수로 받아서 수정하는 경우 이 인자는 공변이 아니라 `불변(invariant)`이 된다.
```csharp
public static void InvariantGeneric(IList<CelestialBody> baseItems)
{
    baseItems[0] = new A { Name = "temp", Mass = 3 };
}
```

## ✅ In 키워드
- in을 사용하면 컴파일러에게 타입 매개변수 입력 위치에서만 사용할 것이라고 알려주게 된다.
- 이는 IComparable을 사용하여 A와 B를 비교할 수 있음을 의미한다.
- 반공변성을 지정한 타입 매개변수는 메서드의 매개변수를 지정하거나 일부 델리게이트의 매개변수를 지정하는 용도로만 사용할 수 있다.
```csharp
public interface IComparable<in T>
{
    int CompareTo(T other);
}
```
## ✅ 델리게이트 매개변수에 대한 공변과 반공변
- `메서드의 매개변수 타입은 반공변(in)이고 반환 타입이 공변(out)이다.`
```csharp
public delegate TResult Func<out TResult>();
```


## ⭐정리⭐
- 가능하면 제네릭 인터페이스나 델리게이트 정의 시에 in, out 데코레이터를 사용하자.
- 가변성과 관련된 에러를 컴파일러가 사전에 확인할 수 있다.
