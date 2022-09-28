---
title: EFC# ITEM 38. 메서드보다 람다 표현식이 낫다.
date: 2022-09-28 21:04:24 +0900
categories: [Effective C#]
tags: [c#]
image:
  path: /assets/img/thumbnail/EffectiveCSharp/ch4.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
---

## ✅ 람다 표현식 동일 코드 반복
```csharp
// 20년 이상 근속자
var earlyFolks = from e in allEmployees
                 where e.Classification == EmployeeType.Salary
                 where e.YearsOfService >= 20
                 where e.MonthlySalary < 4000
                 select e;

// 20년 미만 근속자
var newest = from e in allEmployees
                 where e.Classification == EmployeeType.Salary
                 where e.YearsOfService < 20
                 where e.MonthlySalary < 4000
                 select e;
```
- 앞의 쿼리에서는 where 절을 여러 번에 걸쳐 나누어 썼지만 모든 조건을 결합하여 하나의 where 절로 변경할 수도 있을 것이다. 하지만 이렇게 코드를 수정한다고 해도 성능이 개선되기를 기대하기는 어렵다.

## ✅ 중복되는 람다식 메서드 분리
```csharp
private static bool LowPaidSalaried(Employee e) =>
    e.MonthlySalary < 4000 && e.Classification ==
    EmployeeType.Salary;
    
var earlyFolks = from e in allEmployees
                 where LowPaidSalaried(e) &&
                         e.YearsOfService >= 20
                  select e;
                  
var earlyFolks = from e in allEmployees
                 where LowPaidSalaried(e) &&
                         e.YearsOfService < 20
                  select e;
```
- 좀 단순해진 것 같지만, 분리된 메서드는 재사용될 가능성이 현저히 낫다.


```csharp
private static IQueryable<Employee> LowPaidSalariedFilter
    (this IQueryable<Employee> sequence) =>
      from s in sequence
      where s.Classification == EmployeeType.Salary &&
        s.MonthlySalary < 4000
      select s;
    
var salaried = allEmployees.LowPaidSalariedFilter();

var earlyFolks = salaried.Where(e => e.YearsOfService >= 20);
var newes = salaried.Where(e => e.YearsOfService < 20);
```
- 람다 표현식이 중복적으로 나타나지 않도록 하기 위해서 동일하게 사용되는 로직을 분리하여 호출 체인의 앞쪽으로 이동시켜야 할 수도 있다.

## ⭐정리⭐
- 쿼리를 작성할 때 람다 표현식을 본문으로 하는 조그만 빌딩 블록을 조합하여 완전한 쿼리를 작성할 수 있다.

