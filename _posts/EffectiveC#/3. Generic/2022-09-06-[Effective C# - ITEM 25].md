---
title: EFC# ITEM 25. 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라.
date: 2022-09-06 20:48:55 +0900
categories: [Effective C#]
tags: [c#]
---

`유틸리티 성격의 클래스를 만드는 경우에는 일반 클래스 내에 제네릭 메서드를 작성하는 편이 훨씬 좋다.`

제네릭 클래스는 클래스 전체를 하나의 제약 조건으로 감싸게 되지만, 일반 클래스 내의 제네릭 메서드는 각 메서드마다 개별적은 제약 조건을 설정할 수 있다.

## ✅ 제네릭 클래스 내의 유틸 함수
```csharp
public static class Utils<T>
{
    public static T Max(T left, T right) =>
        Comparer<T>.Default.Compare(left, right) < 0 ? right : left;

    public static T Min(T left, T right) =>
        Comparer<T>.Default.Compare(left, right) < 0 ? left : right;
}

double d1 = 4;
doubld d2 = 5;
double max = Utile<double>.Max(d1,d2);

string foo = "foo";
string bar = "bar";
string sMax = Utils<string>.MAx(foo, bar);
```
- 메서드 호출 시마다 매번 타입 매개변수를 명시적으로 지정해야 한다.
- Math.Max, Math.Min 메서드를 사용할 수 있지만 항상 런타임에 타입 매개변수가 구현한 Comparer를 구현하는지 확인하고 호출해야 한다.

## ✅ 일반 클래스 내의 유틸 함수
```csharp
public static class Utils
{
    public static T Max(T left, T right) =>
            Comparer<T>.Default.Compare(left, right) < 0 ? right : left;

    public static double Max(double left, double right) => //제네릭 메서드
        Math.Max(left, right);

    public static T Min(T left, T right) =>
            Comparer<T>.Default.Compare(left, right) < 0 ? left : right;

    public static double Min(double left, double right) => //제네릭 메서드
        Math.Min(left, right);
}

// 숫자 비교 방법
double d1 = 4;
double d2 = 5;
double max = Utils.Max(d1, d2);

// 문자열 비교 방법
string foo = "foo";
string bar = "bar";
string sMax = Utils.Max(foo, bar);
```
- Min, Max에 대해 오버로딩 메서드를 작성했다.
- `매개변수 타입이 정확히 일치하는 메서드가 구현되어 있으면 그 메서드가 호출되고, 매개변수의 타입이 일치하는 메서드가 없으면 제네릭 메서드가 호출된다.`

> **💡제네릭 클래스를 작성하는 경우**
>
> - 클래스 내 타입 매개변수로 주어진 타입으로 내부 상태를 유지해야 하는 경우.
> - 제네릭 인터페이스를 구현하는 클래스를 만들어야 하는 경우
{: .prompt-info }

## ⭐정리⭐
 - 제네릭 메서드를 구현하여 타입을 컴파일러가 자동으로 찾도록 유도하는 것이 좋다.
