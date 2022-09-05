---
title: EFC# ITEM 24. 베이스 클래스나 인터페이스에 대해서 제네릭을 특화하지 말라.
date: 2022-09-03 21:38:55 +0900
categories: [Effective C#]
tags: [c#]
---

제네릭과 함께 여러 개의 오버로드된 메서드가 있는 경우, 컴파일러가 이 중 하나를 어떻게 선택하는지 정확히 알고 있어야한다. 사용자가 명시적으로 타입을 설정한 일반 메서드보다 제네릭 메서드가 우선적으로 선택되는 경우에 대해서 명확히 이해해야한다.

## ✅ 부모 형식 vs. 제네릭 형식 우선 순위
```csharp
public class MyBase
{
}
 
public interface IMessageWriter
{
    void WriteMessage();
}
 
public class MyDerived : MyBase, IMessageWriter
{
    void IMessageWriter.WriteMessage() =>
        WriteLine("Inside MyDerived.WriteMessage");
}
 
public class AnotherType : IMessageWriter
{
    public void WriteMessage() =>
        WriteLine("Inside AnotherType.WriteMessage");
}
 
class Program
{
    static void WriteMessage(MyBase b)
    {
        WriteLine("Inside WriteMessage(MyBase)");
    }
 
    static void WriteMessage<T>(T obj)
    {
        Write("Inside WriteMessage<T>(T): ");
        WriteLine(obj.ToString());
    }
 
    static void WriteMessage(IMessageWriter obj)
    {
        Write("Inside WriteMessage(IMessageWriter): ");
        obj.WriteMessage();
    }
 
    static void Main(string[] args)
    {
        MyDerived d = new MyDerived();
        WriteLine("Calling Program.WriteMessage");
        WriteMessage(d);
        WriteLine();
 
        WriteLine("Calling through IMessageWriter interface");
        WriteMessage((IMessageWriter)d);
        WriteLine();
 
        WriteLine("Cast to base object");
        WriteMessage((MyBase)d);
        WriteLine();
 
        WriteLine("Another Type test:");
        AnotherType anObject = new AnotherType();
        WriteMessage(anObject);
        WriteLine();
 
        WriteLine("Cast to IMessageWriter:");
        WriteMessage((IMessageWriter)anObject);
    }
}
```

```txt
Calling Program.WriteMessage
Inside WriteMessage<T>(T): Item14.MyDerived
 
Calling through IMessageWriter interface
Inside WriteMessage(IMessageWriter): Inside MyDerived.WriteMessage
 
Cast to base object
Inside WriteMessage(MyBase)
 
Anther Type test:
Inside WriteMessage<T>(T): Item14.AnotherType
 
Cast to IMessageWriter:
Inside WriteMessage(IMessageWriter): Inside AnotherType.WriteMessage
cs
```
1. 첫번째
  - MyBase를 상속한 MyDerived 클래스는 WriteMessage(MyBase b)보다 `WriteMessage<T> (T obj)`에 더 정확히 일치한다.
  - 제네릭 메서드 타입 매개변수인 T를 MyDerived로 대체하면 컴파일러 입장에서는 요청한 메서드와 정확히 일치하는 메서드를 찾을 수 있기 때문이다.
  - 반면 WriteMessage(MyBase b)는 암시적 형변환이 필요하다.

2. 두번째, 세번째
  - 명시적 형변환이 메서드 확인 규칙에 어떻게 영향을 미치는지를 보여준다. 

3. 네번째, 다섯번째
  - 상속 관계는 없지만 특정 인터페이스를 구현하고 있는 타입(IMessageWriter)을 사용할 경우 어떤 메서드가 선택되는지를 보여준다. 
  - 네 번째 경우엔 해당 타입의 매개변수를 필요로 하는 메서드가 없어 제네릭 메서드를 이용하고 마지막은 IMessageWriter 인터페이스로 명시적 형 변환을 해주었기 때문에 IMessageWriter 인터페이스의 매개변수를 가지는 메서드가 호출되었다.


## ⭐정리⭐
- 베이스 클래스와 파생된 클래스에 대해 모두 수행 가능하도록 하기 위해 인터페이스에 대해 제네릭을 특화하면 오류가 발생할 가능성이 높다.
