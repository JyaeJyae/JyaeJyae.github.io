---
title: EFC# ITEM 17. 표준 Dispose 패턴을 구현하라.
date: 2022-08-24 19:39:53 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch2.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---


.Net Framework 내부에서는 비관리 리소스를 정리하는 표준화된 패턴(**Dispose 패턴**)을 사용하고 있다.

## ✅ Dispose Pattern

> 💡 TIP!
> - `비관리 리소스` : 메모리가 아닌 자원을 말하며, 윈도우 핸들, 파일 핸들, 소캣 핸들 등 시스템 자원을 뜻한다.
> - `관리 리소스` : new List<string>()과 같이 메모리처럼 쓰는 자원
> 이런 비관리 리소스들은 가비지 콜렉터가 아닌 개발자가 직접 관리해 줘야 한다.

### 최상위 베이스 클래스
1. 리소스를 정리하기 위해서 IDisposable 인터페이스를 구현해야 한다.
2. 멤버 필드로 비관리 리소스를 포함하는 경우 방어적으로 동작할 수 있도록 finalizer를 추가해야 한다.
3. Dispose와 finalizer는 실제 리소스 정리 작업을 수행하는 다른 가상 메서드에 작업을 위임하도록 작성돼야 한다. 파생 클래스가 고유의 리소스 정리 작업이 필요한 경우 이 가상 메서드를 재정의 할 수 있도록 하기 위함이다.

### 파생 클래스
1. 베이스 클래스에서 정의하고 있는 가상 함수를 반드시 재 호출해야한다.
2. 파생 클래스가 고유의 리소스 정리 작업을 수행해야 한다면 베이스 클래스에서 정의한 가상 메서드를 재정의한다.
3. 멤버 필드로 비관리 리소스를 포함하는 경우에만 finalizer를 추가해야 한다.


## ✅ Finalizer
- 비관리 리소스를 포함하고 있다면 무조건 finalizer를 구현해라.
- 가비지 수집대상으로 체크된 객체에 finalizer가 없다면 즉시 메모리에서 제거된다.
- finalizer를 가진 객체는 `finalizer 큐`에 참조되어 각각의 finalier를 순차적으로 호출하게 된다.
- finalizer가 호출된 객체는 메모리에서 제거될 수 있는 객체로 체크되어 수집되길 기다린다.
- 최초 수집되지 못하였으니 가비지 수집에서 한 세대가 높아지게 되고, `상대적으로 오래 메모리`에 남게 된다.

## ✅ IDisposable Interface
- `비관리 리소스를 정리하는 표준화된 방법`이다. 이를 온전히 구현했다면 최종 사용자에게 적시에 리소스를 해제할 수 있는 메커니즘을 안전하게 제공한다고 할 수 있다. 최종 사용자는 IDisposable을 이용함으로써 finalize과정으로 인해 발생하는 불필요한 배용을 피할 수 있다.

- IDisposable.Dispose() 메서드는 다음 네 가지의 작업을 반드시 수행해야 한다.
  1. 모든 비관리 리소스를 정리한다.
  2. 모든 관리 리소스를 정리한다.
  3. 객체가 정리되었음을 나타내기 위한 플래글 설정. 이 플래그를 통해 이미 정리된 객체에 추가로 정리 작업이 요청되면 ObjectDisposed 예외를 발생시킨다.
  4. finalizer 호출 회피. 이를 위해 `GC.SupperessFinalize(this)`를 호출한다.

## ✅ Virtual Helper Function

파생 클래스의 메모리 해제 후, 베이스 클래스의 메모리 해제 문제를 해결하는 방법이다.
```csharp
protected virtual void Dispose(bool isDisposing);
```
- 상속 받은 클래스에서도 리소스를 해제할 수 있도록 도와주는 코드이다.
- isDisposing 
  - true : 관리/비관리 리소스 모두 해제
  - false : 비관리 리소스만 해제
-  파생 클래스가 추가적인 정리 작업을 수행하는 경우 Dispose(bool) 메서드를 재정의해야 한다.
-  예제.
   ```csharp
   public class MyResource : System.IDisposable
   {
       private bool alreadyDisposed = false;
   
       public void Dispose()
       {
           Dispose(true);
           System.GC.SuppressFinalize(this);
       }
   
       protected virtual void Dispose(bool isDisposing)
       {
           if (alreadyDisposed)
               return;
   
           if(isDisposing)
           {
               // 관리 리소스 정리
           }
   
           // 비관리 리소스 정리
   
           alreadyDisposed = true;
       }
   }
   
   public class DerivedResource : MyResource
   {
       private bool disposed = false; // 자신만의 disposed 플래그
   
       protected override void Dispose(bool isDisposing)
       {
           if (disposed)
               return;
   
           if (isDisposing)
           {
               // 관리 리소스 정리
           }
   
           // 비관리 리소스 정리
   
           base.Dispose(isDisposing); // 베이스 클래스가 GC.SupressFinalize()를 호출한다.
           disposed = true;
       }
   }
   ```

## ✅ 주의점
 - Dispose 메서드는 여러 번 호출하더라도 반드시 동일하게 동작하도록 구현해야 한다.
 - Dispose, finalizer 내에서는 리소스 정리 작업만을 수행하라. 객체의 생명주기와 관련된 문제를 일으킬 수 있다.
 - 클래스가 비관리 리소스를 포함하는 경우에는 반드시 finalizer를 구현해야 하며, 비관리 리소스를 포함하지 않는다면 finalizer를 구현하지 마라

## ⭐정리⭐
- 비관리 리소스를 사용하는 객체는 표준 Dispose 패턴을 사용해야 성능 문제를 막을 수 있다.