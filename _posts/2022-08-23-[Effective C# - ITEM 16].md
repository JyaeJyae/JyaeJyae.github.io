---
title: EFC# ITEM 16. 생성자 내에서는 절대로 가상 함수를 호출하지 말라
date: 2022-08-23
author: JyaeJyae
categories: [Effective C#]
tags: [c#]
---

- 객체가 완전히 생성되기 이전에 가상 함수를 호출하면 이상 동작을 일으킨다.
- 어떤 타입이든 생성자가 수행을 완료할 때까지는 객체가 완전히 생성되었다고 할 수 없다.
- 따라서, 생성자 내에서 가상 함수를 호출하면 예상처럼 동작하지 않는다.

### ✅ 생성자 내에서 가상 함수를 호출하지 말 것.

- 생성자 내에서 가상 함수를 호출하는 예제.

    ```csharp
    class B
    {
        protected B()
        {
            VFunc();
        }
     
        protected virtual void VFunc()
        {
            Console.WriteLine("VFunc in B");
        }
    }
     
    class Derived : B
    {
        private readonly string msg = "Set by Initializer";
     
        public Derived(string msg)
        {
            this.msg = msg;
        }
     
        protected override void VFunc()
        {
            Console.WriteLine(msg);
        }
     
        public static void Main()
        {
            var d = new Derived("Constructed in main");
        }
    }
    ```
    - 위 코드에서 콘솔에 어떤 문자열이 출력 될 것인가?
    - 👉 답 : "Set by initializer"
      1. 부모 클래스(B)의 생성자 호출
      2. 부모 생성자 내에서 가상 함수 호출
      3. 자식(Derived)의 오버라이딩된 함수 호출
      4. 💡**이 시점에서 자식 클래스의 생성자가 완료되지 않았지만 자식 클래스의 함수 호출**

- 수정된 베이스 클래스 예제.

    ```csharp
    abstract class B
    {
        protected B()
        {
            VFunc();
        }
     
        protected abstract void VFunc();
    }
     
    class Derived : B
    {
        private readonly string msg = "Set by Initializer";
     
        public Derived(string msg)
        {
            this.msg = msg;
        }
     
        protected override void VFunc()
        {
            Console.WriteLine(msg);
        }
     
        public static void Main()
        {
            var d = new Derived("Constructed in main");
        }
    }
    ```
    
    - 이 경우, B타입의 객체를 생성할 수 없기 때문에 파생 클래스에서 반드시 VFunc()를 구현해야 정상적으로 컴파일 된다.
    - 이러한 설계 방법이 생성자 내에서 추상 함수를 호출했을 때 런타임 예외를 피할 수 있는 유일한 해법이다.

### ⭐정리⭐

- 베이스 클래스 생성자 내에서 가상 함수를 호출하면 파생 클래스가 가상 함수를 어떻게 구현했는지에 따라 매우 민감하게 동작하기 때문에 호출하지 말자.
- 자식 클래스의 생성자는 부모 클래스의 기본 생성자 이후에 호출된다.
