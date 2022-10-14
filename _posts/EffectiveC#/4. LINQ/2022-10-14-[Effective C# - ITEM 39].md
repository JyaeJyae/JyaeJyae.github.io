---
title: EFC# ITEM 39. function과 action 내에서는 예외가 발생하지 않도록 하라.
date: 2022-10-14 21:14:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: EFC# ITEM 39. function과 action 내에서는 예외가 발생하지 않도록 하라.
---

일련의 값을 순차적으로 처리하는 코드가 중간 어디쯤에서 예외를 일으키면 상태를 복구할 수 없는 문제에 봉착한다. 처리가 완료된 요소가 몇 개인지를 알 수 없고 따라서 원복해야 할 요소가 무엇인지도 알 수 없다.

## ✅ 예상 가능한 오류
```csharp
allEmployees.FindAll(
    e => e.Classification == EmployeeType.Active).
    ForeEach(e => e.MonthlySalary *= 1.05M);
```
- 람다 표현식으로 나타낸 액션 메서드가 절대 예외를 발생시키지 않도록 하는 것이다.
- 단순희 필터링하는 것만으로도 예외가 발생하는 것을 피할 수 있다.
- 하지만, 때로는 예외가 발생하지 않도록 코드를 작성하는 것이 불가능한 경우도 있다.


## ✅ 예상 불가능한 오류
```csharp
var updates = (from e in allEmployees
               select new Employee
               {
                   EmployeeID = e.EmployeeID,
                   Classification = e.Classification,
                   YearsOfService = e.YearsOfService,
                   MonthlySalary = e.MonthlySalary *= 1.05M
               }).ToList();
```
- `원본 시퀀스의 복사본을 마련해두고 이 복사본에 대해 알고리즘을 적용한 후 모든 작업이 정상적으로 완료된 경우에만 원본 시퀀스를 대체하는 방식`이 있다.
- 이 경우 코드의 양이 늘어나며 성능에도 영향을 미칠 수 있지만, 기존의 데이터를 수정하는 대신 새로운 데이터를 생성하도록 작성한다면 작업 완료를 확인할 기회가 생길 뿐만 아니라 문제가 생길 경우 프로그램의 상태를 변경하지 않기에 안전하다.


## ⭐정리⭐
- 람다 표현식을 사용하는 경우 그 내부에서 예외가 발생하면 문제의 원인을 규명하기가 매우 어려워 진다.
- 필요한 작업을 복사본에 대해서 수행하고, 예외가 발생하지 않은 경우에 한해서 전체 시퀀스를 변경하는 방식은 상당히 유용하다.

