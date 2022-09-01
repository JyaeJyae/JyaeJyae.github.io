---
title: EFC# ITEM 21. 타입 매개변수가 IDisposable을 구현한 경우를 대비하여 제네릭 클래스를 작성하라.
date: 2022-08-30 19:54:55 +0900
categories: [Effective C#]
tags: [c#]
---

> 👉 제약 조건의 역할
> 1. 런타임 오류가 발생할 수 있는 부분을 컴파일 타임 오류로 대체한다.
> 2. 타입 매개변수로 사용할 수 있는 타입을 명확히 규정하여 사용자에게 도움을 준다.

제약 조건은 타입 매개변수가 무엇을 해야 하는지만을 규정할 수 있고, 무엇을 해서는 안 되는지를 정의할 수 없다.

하지만, 타입 매개변수로 지정하는 타입이 IDisposable을 구현하고 있다면 특별한 추가 작업이 필요하다.

## ✅ Method 내에서 타입 객체를 생성한 경우
```csharp
public interface IEngine
{
    void DoWork();
}

public class EngineDriverOne<T> where T : IEngine, new()
{
    public void GetThingsDone()
    {
        T driver = new T();
        driver.DoWork();
    }
}
```
- T가 IDisposable을 구현한 타입일 경우 리소스 누수가 발생할 수 있다.
- 따라서, T 타입으로 지역변수를 생성할 때마다 T IDisposable을 구현하고 있는지 확인해야 하며, 만약 IDisposable을 구현하고 있다면 추가적인 처리를 해야한다.

```csharp
public void GetThingsDone()
{
    T driver = new T();
    using(driver as IDisposable)
    {
        driver.DoWork();
    }
}
```
- 컴파일러는 IDisposable로 형 변환된 객체를 저장하기 위해서 숨겨진 지역변수를 생성한다.
- T가 IDisposable를 구현하지 않았다면, 이 지역 변수의 값은 null이되고 Dispose()가 호출되지 않는다.
- T가 IDisposable을 구현했다면 `using 블록을 종료할 때 Dispose() 메서드가 호출`된다.

## ✅ Class 내에서 `멤버 변수`로 타입 객체를 선언한 경우
```csharp
public sealed class EngineDriver2<T> : IDisposable where T : IEngine, new()
{
    // 생성 작업이 오래 걸릴 수도 있으므로, Lazy를 이용하여 초기화 진행
    private Lazy<T> driver = new Lazy<T> (() => new T());

    public void GetThingsDone() => driver.Value.DoWork();

    // IDisposable 멤버
    public void Dispose()
    {
         if (driver.IsValueCreated)
        {
            var resource = driver.Value as IDisposable;
            resource?.Dispose();
        }
    }
}
```
- IDisposable을 구현했을 가능성이 있는 타입으로 멤버 변수를 선언한 것이기 때문에 제네릭 클래스에서 IDisposable을 구현하여 해당 리소스를 처리해야한다.
- Sealed 클래스로 선언하면 표준 Dispose 패턴을 모두 구현할 필요가 없다. 하지만, 새로운 파생 클래스를 사용할 수 없다.
- 객체의 소유권을 제네릭 클래스 외부로 옮기면 new() 제약 조건을 제거할 수 있다.
  ```csharp
  public sealsed class EngineDriver<T>   where T : IEngine
  {
      private T driver;
      
      public EngineDriver(T driver)
      {
          this.driver = driver;
      }
      
      public void GetThingsDone()
      {
          driver.DoWork();
      }
  }
  ```

## ⭐정리⭐
- 제네릭 클래스의 타입 매개변수로 객체를 생성하는 경우, 이 타입이 IDisposable을 구현하고 있는지 확인해야 한다.
-  타입 매개변수로는 지역변수 정도만 생성하는 것이 좋다.
-  타입 매개변수로 멤버 변수를 선언해야 하는 경우 지연 생성을 사용하거나 제네릭 클래스에서 IDisposable을 구현해야 할 수 있다.
