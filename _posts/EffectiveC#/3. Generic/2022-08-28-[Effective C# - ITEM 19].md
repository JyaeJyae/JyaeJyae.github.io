---
title: EFC# ITEM 19. 런타임에 타입을 확인하여 최적의 알고리즘을 사용하라.
date: 2022-08-28 20:50:55 +0900
categories: [Effective C#]
tags: [c#]
---

제네릭을 활용하면 공동 메서드를 작성할 수 있기 때문에 코드의 양이 줄어들어 유용하다.

하지만, 타입이나 메서드를 제네릭화하면 구체적인 타입이 주는 장접을 잃고 타입의 세부적인 특성까지 고려하여 최적화한 알고리즘을 사용할 수 없게 된다.

만약, 어떤 알고리즘이 특정 타입에 대해 더 효율적으로 동작한다고 생각된다면 그냥 그 타입을 이용하도록 코드를 작성하라.

## ✅ 비효율적인 제네릭 사용
- 특정 타입의 시퀀스를 역순으로 순회하기 위한 클래스 `ReverseEnumerable<T>`
  ```csharp
  public sealed class ReverseEnumerable<T>   : IEnumerable<T>
  {
      private class ReverseEnumerator :    IEnumerator<T>
      {
          int currentIndex;
          IList<T> collection;
  
          public ReverseEnumerator   (IList<T> srcCollection)
          {
              collection = srcCollection;
              currentIndex = collection.   Count;
          }
  
          // IEnumerator<T> 멤버
          public T Current => collection   [currentIndex];
  
          // IDisposable 멤버
          public void Dispose()
          {
              // 세부 구현 내용은 생략했으나   반드시 구현해야 한다.
              // 왜냐하면 IEnumerator<T>는   IDisposable을 상속하고 있기   때문이다.
              // 이 클래스는 sealed    클래스로 선언되었으므로    protected Dispose() 메서드는   필요 없다.
          }
  
          // IEnumerator 멤버
          object System.Collections.   IEnumerator.Current => this.   Current;
          public bool MoveNext() =>    --currentIndex >= 0;       
          public void Reset() =>   currentIndex = collection.Count;
      }
  
      IEnumerable<T> sourceSequence;
      IList<T> originalSequence;
  
      // 생성자
      public ReverseEnumerable   (IEnumerable<T> sequence)
      {
          sourceSequence = sequence;
      }
  
      // IEnumerable<T> 멤버
      public IEnumerator<T> GetEnumerator()
      {
          // 역순으로 순회하기 위해서 원래 시퀀스를 복사한다.
          if(originalSequence == null)
          {
              originalSequence = new   List<T>();
              foreach(T item in    sourceSequence)
                  originalSequence.Add   (item);
          }
           
          return new ReverseEnumerator   (originalSequence);
      }
  
      // IEnumerable 멤버
      System.Collections.IEnumerator   System.Collections.IEnumerable.   GetEnumerator() => this.GetEnumerator  ();
  }
  ```
  - 타입 매개변수에 대한 가정이 거의 없다.
  - 랜덤 엑세스를 지원하지 않는 컬렉션에 대해서 개별 요소를 역순으로 순회하기 위한 유일한 방법이다.
  - 하지만, 대부분의 컬렉션들이 랜덤 엑세스를 지원하기 때문에 이와 같은 코드는 상당히 비효율적이다.
  - 생성자로 전달한 인자가 IList<T>를 지원한다면 이처럼 복제본을 만들 이유가 없다.

## ✅ 타입 매개변수 생성자 추가
```csharp
public ReverseEnumerable(IEnumerable sequence)  
{  
sourceSequence = sequence;

// 만약 sequence가 IList<T>를 구현하지 않았다면
// originalSequence가 null이 되지만
// 문제되지 않는다.
originalSequence = sequence as IList<T>;

}

public ReverseEnumerable(IList sequence)  
{  
sourceSequence = sequence;  
originalSequence = sequence;  
}
```
- IList<T>를 사용하면 IEnumerable<T> 만을 사용할 때보다 매개 변수 IList<T> 타입인 것을 컴파일 타임에 알 수 있으므로 구현이 더 용이하다.
- 하지만, 어떤 객체는 런타임 시에는 IList<T> 타입의 객체라 하더라고 컴파일 타임에는 IEnumberable<T> 타입으로 간주될 수 있다.

## ✅ 런타임에 확인
```csharp
public ReverseEnumerable(IEnumerable sequence)  
{  
sourceSequence = sequence;

// 만약 sequence가 IList<T>를 구현하지 않았다면
// originalSequence가 null이 되지만
// 문제되지 않습니다.
originalSequence = sequence as IList<T>;

}

public ReverseEnumerable(IList sequence)  
{  
sourceSequence = sequence;  
originalSequence = sequence;  
}
```
- 생서자 오버로드 외에도 런타임 타입을 확인하도록 코드를 작성해야 한다.
- 하지만 IList<T>를 구현하지 않고 ICollection<T>만을 구현한 컬렉션들에 대해서는 여전히 비효율적이다.

## ✅ 속도 개선
```csharp
// IEnumerable 멤버  
public IEnumerator GetEnumerator()  
{  
// string은 매우 특별한 경우입니다.  
if(sourceSequence is string)  
{  
// 컴파일타임에 T는 char가 아닐 것이므로  
// 캐스트에 주의해야 합니다.  
return new ReverseEnumerator(sourceSequence as string) as IEnumerator;  
}

// 역순으로 순회하기 위해서
// 원래 시퀀스를 복사한다.
if (originalSequence == null)
{
    if (sourceSequence is ICollection<T>)
    {
        ICollection<T> source = sourceSequence as ICollection<T>;
        originalSequence = new List<T>(source.Count);
    }
    else
    {
        originalSequence = new List<T>();
    }

    foreach (T item in sourceSequence)
        originalSequence.Add(item);
}
return new ReverseEnumerator(originalSequence);

}
```
- 입력 시퀀스 ICollection<T>만을 구현한 경우 입력 시퀀스에 대해 본제본을 생성해야 하므로 매우 느리게 동작할 수 밖에 없다.
- ICollection<T>가 제공하는 Count 속성을 활용하여 저장소 공간을 미리 초기화하도록 코드를 개선한 예이다.
- ReverseEnumerable<T> 내에서 수행되는 매개변수에 대한 테스트는 `모두 런타임`에 이뤄진다는 것을 유념해야한다.
즉, 추가 기능을 확인하는 과정도 일정 부분 비용이 발생하지만 대부분의 경우에는 이 비용은 모든 요소를 복사하는 것에 비해 훨씬 적다.

## ⭐정리⭐
- 타입에 대한 제약 조건을 거의 사용하지 않으면서도 타입 매개변수로 지정될 가능성이 있는 타입들의 고유한 특성을 고려하고 특화된 기능들을 최대한 활용하여 제네릭 타입을 만드는 방법을 살펴보았다.
- 이와 같이 하면 재사용성이 높으면서도 개별 타입에 최적화된 코드를 작성할 수 있다.

