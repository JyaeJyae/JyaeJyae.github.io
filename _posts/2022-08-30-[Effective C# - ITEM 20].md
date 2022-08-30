---
title: EFC# ITEM 20. IComparable<T>와 IComparer<T>를 이용하여 객체의 선후 관계를 정의하라.
date: 2022-08-30 20:53:55 +0900
categories: [Effective C#]
tags: [c#]
---

.NET Framework는 객체의 선후 관계를 정의하기 위해서 `IComparable<T>`와 `IComparer<T>` 2개의 인터페이스를 제공한다.

- IComparable<T> :  타입의 기본적인 선후 관계를 정의
- IComparer<T> : 기본적인 선후 관계 이외에 추가적인 선후 관계 정의

> 👉 이번 항목에서 우리가 배울 것!
> 1. 객체의 선후 관계를 구현하는 방법
> 2. 인터페이스를 이용하여 .NET 프레임워크가 객체의 순서를 정렬하는 원리 

## ✅ IComparable, IComparable<T>
- `현재 객체가 대상 객체보다 작으면 0보다 작은 값을, 같으면 0을, 크면 0을 리턴하는 함수이다.`
  ```csharp
  public interface IComparable
  {
      int CompareTo(object obj);
  }
  ```

  ```csharp
    public struct Customer :IComparable<Customer>,IComparable
    {
        private readonly string name;

        public Customer(string name)
        {
            this.name = name;
        }

        // IComparable<Customer> 멤버
        public int CompareTo(Customer other) => name.CompareTo(other.name);

        // IComparable 멤버
        int IComparable.CompareTo(object obj)
        {
            if (!(obj is Customer))
                throw new ArgumentException("Argument is not a Customer", "obj");

            Customer other = (Customer)obj;

            return this.CompareTo(other);
        }
    }
  ```
  - 타입 매개변수를 취하지 않는 IComparable 단점
    - Interface를 구현하기 위해 매개변수에 대한 타입을 런타임에 확인해야 한다.
    - 비교를 위해 박싱/언박싱이 필요하므로 비교할 쌔마다 상당한 성능 비용이 발생한다.
  - IComparable 구현 시 중요한 점
    - 명시적으로 인터페이스를 구현하고 추가적으로 강력한 타입의 public 오버로드 메서드를 함께 구현해야 한다.

## ✅ IComparer, IComparer<T>
- `x가 y보다 작다면 0보다 작은 값을, 같으면 0, 크면 0보다 큰 값을 반환한다.`
```csharp
public interface IComparer
{
    int Compare(object x, object y);
}
```
- 관계 연산자 재정의
  - C#은 표준 관계 연산자를 오버로딩할 수 있으므로 구체적으로 타입을 취하는 CompareTo() Method를 사용하여 `오버로딩`하는 것이 좋다.


  ```csharp
  public struct Customer :   IComarable<Customer>, IComparable
  {
      private readonly string name;
      public Customer(string name) {
          this.name = name;
      }
  
      // IComparable<Customer> 멤버
      public int CompareTo(Customer other) => name.CompareTo(other.name);
  
      // IComparable 멤버
      int IComparable.CompareTo(object obj) {
  
          if (!(obj is Customer))
  
              throw new ArgumentException("Argument is not a Customer", "obj");
  
          Customer otherCustomer = (Customer)obj;
  
          return this.CompareTo(otherCustomer);
  
      }
  
     // 관계 연산자
     public static bool operator <(Customer left, Customer right) => left.CompareTo(right) < 0;
     public static bool operator <=(Customer left, Customer right) => left.CompareTo(right) <= 0;
     public static bool operator >(Customer left, Customer right) => left.CompareTo(right) > 0;
     public static bool operator >=(Customer left, Customer right) => left.CompareTo(right) >= 0;
  
  }
  ```
  
## ✅ Custom 선후 관계 연산자 정의

> 💡 delegate int Comparison<T> 
> - 선후 관계 연산을 대신하는 대리자가 존재하는데, int Comparison<T>는 제너릭 타입 T에 대해 두 인스턴스를 비교하여 int를 반환하는 대리자이다.

- IComparer<T>
  ```csharp
  // 매출을 기반으로 Customer 객체를   비교하는 클래스
  // 이 클래스는 항상 인터페이스 참조를   통해서 사용된다.
  private class RevenueComparer :   IComparer<Customer>
  {  
	int IComparer<Customer>.Compare(Customer left, Customer right)  
		=> left.revenue.CompareTo(right.revenue);
  }
  ```


### ⭐정리⭐
- 참조타입의 선후 관계를 따질 때는 객체의 내용을 기반으로 하지만 동일성 비교는 두 객체의 참조만을 비교한다. Equal()가 false를 반환하더라도 CompareTo()는 0을 반환할 수 있다. 동일성 비교와 선후 관계 비교는 항상 함께 제공되어야 한다.
- IComparable, IComparer는 타입의 선후 관계를 제공하기 위한 표준 메커니즘이다. 기본적인 선후 관계는 IComparable을 통해 구현해야한다. IComparable을 구현할 때에는 관계 연산자도 함께 오버로딩하여 일관된 결과를 제공해야한다.
- IComparer를 이용하면 추가적인 선후 관계를 정의할 수 있을 뿐 아니라 우리가 직접 개발하지 않는 타입에 대해서도 임의의 선후 관계를 추가로 정의할 수 있다.
