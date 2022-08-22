---
title: EFC# ITEM 15. 불필요한 객체를 만들지 말라
date: 2022-08-22 20:50:32 +0900
categories: [Effective C#]
tags: [c#]
---


- Garbage Collector(GC) 는 사용자를 대신하여 메모리를 관리하며 사용하지 않는 객체를 효율적인 방식으로 제거한다.
- 하지만, Heap에서 새로운 객체를 생성하고 삭제하는 작업은 생각보다 많은 프로세서 시간을 사용하고 너무 많은 객체를 생성하면 심각한 성능 저하 문제 초래한다.
- 따라서, Garbage Collector(GC)가 과도하게 동작하지 않도록 주의해야 한다.



### ✅ 자주 사용하는 지역변수를 멤버 변수로 변경

자주 호출되는 이벤트 핸들러에서 참조 타입의 객체를 지역 변수로 매번 생성하면, Garbage Collector(GC)가 객체를 생성, 제거를 반복해 성능에 문제를 일으킬 수 있다.

- Window Pait 이벤트 핸들러 내에서 GDI 객체 할당하는 예제.

    ```csharp
        protected override void OnPaint(PaintEventArgs e)
        {
            // 나쁜 예, Paint 이벤트가 발생할 때마다 동일한 폰트를 생성한다.
            using (Font MyFont = new Font("Arial", 10.0f))
            {
                e.Graphics.DrawString(DateTime.Now.ToString(),
                    MyFont, Brushes.Black, new PointF(0, 0));
            }
    
            base.OnPaint(e);
        }
    ```
    - 이 이벤트 핸들러가 호출될 때마다 동일한 Font 객체를 매번 다시 생성한다.

- Font 객체를 지역 변수로 선언할 것이 아니라 멤버 변수로 변경하여 폰트 객체를 한번만 생성한 후 이를 재사용 하도록 개선해야 한다.

    ```csharp
        public Font MyFont { get; private set; }
    
        protected override void OnPaint(PaintEventArgs e)
        {
            e.Graphics.DrawString(DateTime.Now.ToString(),
                MyFont, Brushes.Black, new PointF(0, 0));
    
            base.OnPaint(e);
        }
    ```

    - 이처럼 코드를 수정하면 Paint 이벤트가 발생할 때마다 새로운 Garbage가 생성되지 않으므로 Garbage Collector(GC)가 해야 하는 일의 양 감소된다.
    - Font 타입과 같이 IDisposable 인터페이스를 구현한 타입의 객체를 `지역변수에서 멤버변수로 변경하면 이 클래스도 반드시 IDisposable을 구현해야한다.` 

> **💡TIP!**
>
> - 자주 호출되는 Method 내에서 참조 타입(값 타입 제외)의 `객체를 매번 생성하는 경우라면 지역변수를 멤버 변수로 변경하는 것이 좋다.`
> - 하지만 호출 빈도가 그리 빈번하지 않은 경우라면 굳이 변경할 필요가 없다.
> - 지역 변수를 모조리 멤버 변수로 변경하여 동일할 객체를 생성하는 것을 피하라는 것이 아니다.



### ✅ Dependency Injection을 활용하면 자주 사용되는 객체를 Static 멤버 변수로 선언하여 재사용

종속성 삽입(Dependency Injection)을 활용하여 자주 사용되는 객체를 생성했다가 이를 재활용하는 방법이다.

- Bruches.Black 예제 : 유사한 객체를 반복적으로 할당하는 것을 피하는 기법

  ```csharp
  public static Brush Black
  {
      get
      {
          if (blackBrush == null)
          {
              blackBrush = new SolidBrush(Color.Black);
          }
  
          return blackBrush;
      }
  }
  ```

  - 지역 변수로 매번 새로운 검정 브러시를 생성하면 엄청난 수의 검정 브러시 생성될 것이고 이후 모두를 삭제해야 한다.(X)
  - 멤버 변수로 변경한다면, 프로그램이 수행되는 동안 수십 개의 창과 컨트롤들이 생성될 때마다 수 십개의 검정 브러시 생성할 것이다.(X)
  - 필요할 때마다 재사용할 수 있는 검정 브러시 생성(O)

- `브러시가 최초로 요청했을 때 비로소 객체를 생성한다. 이렇게 생성된 검정 브러시를 저장해두고 동일한 요청이 있을 때 마다 이 객체를 돌려준다.`

- 👎단점

  - 생성된 객체가 메모리상에 필요 이상으로 오랫동안 남아 있을 수도 있다.

  - Dispose() 메서드를 호출해야 할 시점을 결정할 수 없기 때문에 비관리 리소스를 삭제할 수 없다.

    

### ✅ Immutable(변경 불가능한) 타입

변경 불가능할 대표적인 예로는 System.String이 있다. string 객체가 생성되면 이 객체가 가지고 있는 문자열의 내용은 수정이 불가능하다. 다만, 새로운 문자열을 가진 새로운 string 객체가 다시 생성되는 것이다.

- string 예제.

  ```csharp
  // 이렇게 코드를 작성하면...
  public static void Main(string[] args)
  {
      string msg = "Hello, ";
      msg += thisUser.Name;
      msg += ". Today is";
      msg += System.DateTime.Now.ToString();
  }
  
  
  // 실제로는 매우 비효율적으로 작업이 이뤄진다.
  string msg = "Hello, ";
    
  string tmp1 = new String(msg + thisUser.Name);
  msg = tmp1; // "Hello, "는 가비지가 된다.
    
  string tmp2 = new String(msg + ". Today is ");
  msg = tmp2; // "Hello, <user>"는 가비지가 된다.
    
  string tmp3 = new STring(msg + DateTime.Now.ToString());
  msg = tmp3; // "Hello, <user>. Today is "는 가비지가 된다.
  ```

  - string 클래스 내의 `"+=" 연산자는 기존 문자열에 새로운 문자열을 더하는 것이 아니라 완전히 새로운 string 객체를 생성하여 반환합니다.`

- **문자열 보간** 사용 예제.

  ```csharp
  string msg = string.Format("Hello, {0}. Today is {1}", thisUser.Name, DateTime.Now.ToString());
  ```

  - 문자열 보간을 사용하면 훨씬 간단히 코드를 작성할 수 있다.

- **StringBuilder** 클래스 사용 예제.

  ```csharp
  // 문자열 생성 과정이 복잡하면 StringBuilder를 사용
  
  StringBuilder msg = new StringBuilder("Hello, ");
    
  msg.Append(thisUser.Name);
  msg.Append(". Today is ");
  msg.Append(DateTime.Now.ToString());
    
  string finalMsg = msg.ToString(); 
  ```

  - 실제로 수정 가능한 문자열을 나타내기 위한 타입으로, 새로운 문자열을 생성하거나 수정, 변경 등의 작업을 수행할 수 있으며, 최종적으로 변경 불가능한 string 타입의 객체를 가져오는 기능을 제공한다.



### ⭐정리⭐

- 객체를 과도하게 생성하는 것을 피하고 불필요한 객체를 생성하지 마라.
- 메서드 내에서 참조 타입의 객체를 만드는 것을 피하고 대신, 지역변수를 멤버변수로 변경하거나 자주 사용하는 인스턴스를 정적 객체로 변경하는 것을 고려해라.
- 변경 불가능한 타입을 작성하는 경우 이에 대응하는 변경 가능한 빌더 클래스를 같이 작성해라.
