---
title: EFC# ITEM 35. 확장 메서드는 절대 오버로드하지 마라.
date: 2022-09-26 21:15:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

> **💡확장 메서드를 작성해야 하는 이유**
>
> 1. 인터페이스에 기본 구현 추가
> 2. 닫힌 제네릭 타입에 동작 추가
> 3. 조합 가능한 인터페이스 작성
{: .prompt-info }

확장 메서드를 이용하여 인터페이스에 대한 기본 구현체를 제공하는 것이 괜찮은 방법인 것은 분명하지만 타입을 확장하기 위한 용도로 사용하는 것은 적절하지 않으며 더 좋은 대안들이 많다.

확장 메서드를 과도하게 사용하거나 잘못 사용하면 메서드 충돌로 인해 유지보수 비용이 급격히 증가하는 곤격에 처할 수 있다.

## ✅ 여러 네임 스페이스에 걸쳐 선언된 확장 메서드
```csharp
namespace ConsoleExtensions
{
    public static class ConsoleReport
    {
        public static string Format(this Person target) =>
            $"{target.LastName, 20} {target.FirstName, 15}";
    }
}

namespace XmlExtensions
{
    public static class XmlReport
    {
        public static string Format(this Person target) =>
            new XElement("Person",
                new XElement("LastName", target.LastName),
                new XElement("FirstName", target.FirstName)
                ).ToString();
    }
}
```
- **문제점**
  - 1. 개발자가 실수로라도 네임스페이스를 잘못 참조하게 되면 프로그램의 동작 방식이 바뀌어 버린다.
  - 2. 확장 메서드가 포함된 네임스페이스를 참조하는 것을 잊어버리면 프로그램은 컴파일조차 되지 않는다.
- `확장 메서드는 여러 네임스페이스에 오버로드 해서는 안된다.`
- 동일한 확장 메서드를 여러개 만들어야 한다면 이름을 달리하고 정적 메서드로 작성하는 것을 고려하자.

## ⭐정리⭐
- 확장 메서드는 논리적으로 추가하려는 기능이 타입 내에 포함되는 것이 적절한 경우에만 사용하자.

