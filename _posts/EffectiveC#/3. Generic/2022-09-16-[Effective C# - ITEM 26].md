---
title: EFC# ITEM 26. 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라.
date: 2022-09-16 21:00:55 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/3장.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: EFC# ITEM 26. 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라.
---

새로운 라이브러리를 개발할 때 제네릭 타입뿐 아니라 고전적인 방식도 함께 지원한다면 라이브러리의 활용도를 좀 더 높일 수 있다.

## ✅ 제네릭 방식
```csharp
public class Name : IComparable<Name>, IEquatable<Name>
{
    public string First { get; set; }
    public string Last { get; set; }
    public string Middle { get; set; }

    public int CompareTo(Name other)
    {
    	if (Object.ReferenceEquals(this, other))
        	return 0;
        if (Object.ReferenceEquals(other, null))
        	return 1;
        
        int rVal = Comparer<string>.Default.Compare(Last, other.Last);
        
        if (rVal != 0)
        	return rVal;
        
        rVal = Comparer<string>.Default.Compare(First, other.First);
        if (rVal != 0)
        	return rVal;
        
        return Comparer<string>.Default.Compare(Middle, other.Middle);
    }
    
    public bool Eqauls(Name other)
    {
    	if (ReferenceEquals(this, other))
       		return true;
        if (ReferenceEquals(other, null))
        	return false;
            
        return Last == other.Last &&
        	First == other.First &&
            Middle == other.Middle;
    }
}
```
- 서로 다른 타입 간의 동일성 체크 작업이 없다. => 제네릭 메서드로 정의

## ✅ 제네릭 메서드
```csharp
public static bool CheckEquality<T>(T left, T right)
	where T : IEquatable<T>
{
    if (left == null)
        return right == null;

    return left.Equals(right);
}
```
- IEquatable<Name>.Equals()가 아닌 system.Object.Equals() 호출한다.
- System.Object.Equals()는 Object 타입을 취하지만, IEqualtable<T>.Equals()는 T 타입을 매개변수로 취하기 때문이다.
- 따라서 논제네릭 클래스들을 위해서도 따로 타입 동일성 비교 함수를 구현해주어야 한다.
```csharp
public override bool Equals(object obj)
{
    if (obj.GetType() == typeof(Name))
    	return this.Equals(obj as Name);
    else
    	return false;
}
```

## ✅ 논제네릭 방식
```csharp
public class Name : IComparable<Name>, IEquatable<Name>, IComparable
{
    int IComparable.CompareTo(object obj)
    {
    	if (obj.GetType() != typeof(Name))
        	throw new ArgumentException("Argument is not a Name object");
        
        return this.CompareTo(obj as Name);
    }
}
```
- IComparable 인터페이스도 함께 구현해야한다.
- 제네릭 인터페이스를 구현한 메서드가 우선적으로 선택될 것이다.

> **💡컬렉션 인터페이스의 상속 관계**
>
> - IEnumerable<T>는 IEnumerable을 상속
> - ICollection<T>는 ICollection을 상속받지않음
> - IList<T>는 IList를 상속받지 않음
> - IList<T>와 ICollection<T>는 IEnumerable<T>를 상속
{: .prompt-info }

## ⭐정리⭐
- 예전에 개발된 코드와 함께 사용되야한다면 논 제네릭 인터페이스를 구현하라.
- 또한 인터페이스를 명시적으로 구현하여 개발자가 실수로 새로운 코드에 논제네릭을 사용하지 않도록 하라.
