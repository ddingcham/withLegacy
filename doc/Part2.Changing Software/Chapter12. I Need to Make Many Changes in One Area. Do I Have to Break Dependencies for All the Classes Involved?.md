# 클래스 의존 관계, 반드시 없애야 할까?
I Need to Make Many Changes in One Area. Do I Have to Break Dependencies for All the Classes Involved?  

변경 지점들이 좁은 범위 내에서 여기저기 흩어져 있는 경우  
// 가장 골치 아픈 케이스  
다수의 변경을 모아서 한 번에 테스트 루틴을 작성할 수 있는 위치를 찾으려면  
#### 상위 수준에서의 테스트
// 한 걸음 물러서기  

* Ex 1] 다수의 private 메소드 변경에 대한 테스트를 한 개의 public 메소드에 대한 테스트 루틴으로  
* Ex 2] 다수 객체 간의 협업 동작에 대한 테스트를 한 개 객체의 인터페이스에 대한 테스트 루틴으로  
#### 상위 수준에서의 테스트는 중요한 도구지만,
#### 단위 테스트를 대체하는 건 아니다.
#### 단위 테스트를 작성하기 위한 첫 단계일 뿐이다.

## 교차 지점 (Interception Point)
특정 변경에 의한 영향을 감지할 수 있는 프로그램 내의 위치  
> 심도 있는 추론 및 다수의 의존 관계 제거가 필요하다.  

### 교차 지점의 판단
* 변경이 필요한 위치들을 확인한다.  
* 이 지점들이 외부에 미치는 영향을 추적한다.  
* 영향이 탐지되는 모든 위치가 교차 지점이다.  
* **최상의 교차 지점은 전체적인 과정을 통해 판단할 수 있다.**  

### Ex] Invoice 클래스에서의 금액 계산 방법 변경
#### 선택과 집중에 주목
```
public class Invoice
{
  ...
  public Money getValue() {
    Money = total = itemsSum();
    if (billingDate.after(Date.yearEnd(openingDate))) {
      if (originator.getState().equals("FL") ||
          originator.getState().euqals("Ny"))
        total.add(getLocalShipping());
      else
        total.add(getDefaultShipping());
    }
    else
      total.add(getSpanningShipping());
    total.add(getTax());
    return total;
  }
}
```

#### 뉴욕(NY)으로 보내지는 운송 비용의 계산 방법을 변경해야 한다면?  
출하 비용 계산 로직에 대한 책임을 ShippingPricer라는 새로운 클래스에게 위임  
```
public Money getValue() {
  Money total = itemsSum();
  total.add(shippingPricer.getPrice());
  total.add(getTax());
  return total;
}
```
* getValue 메소드가 수행했던 작업 대부분이 위임  
* 송장 날짜를 알고 있는 전문가 ShippingPricer 생성을 위해 Invoice 생성자도 변경 필요  

### 교차 지점 찾기  
* 변경 위치에서부터 영향 추적을 시작한다.  
  * getValue() 메소드는 다른 결과를 낳는다.  
    > BillingStatement.makeStatement() ---> Invoice.getValue()  
  * Invoice의 생성자 수정  
    > 생성자에 의존하는 코드도 살펴봐야 한다.  
  * ShippingPricer 생성자는 Invoice를 제외하고는 아무 것에도 영향을 미치지 않는다.  
    > 현재는 Client가 Invoice 뿐이다.  

* 교차 지점 후보군 분석  
  > 영향 추적이 가능한 변경 사항  
    >> : 교차 지점으로 적합한 부분  
    >> : 특정 변경에 의한 영향을 감지할 수 있는 프로그램 내의 위치  
  * Invoice.Constructor  
    > 영향 추적이 가능한 변경 사항  
  * ShippingPricer  
    > ShippingPricer는 Invoice에 캡슐화 되어 있으므로  
    > 우리가 직접 접근할 수 있는 시나리오가 없다.  
    > **Invoice.getValue()** 자체를 확인할 수는 없다.  
  * Invoice.getValue()  
    > 영향 추적이 가능한 변경 사항  
  * BillingStatement.makeStatement()  
    > 영향 추적이 가능한 변경 사항 이지만....  
    > 부가적인 작업량이 늘어날 수 있다.  
    > 우리가 테스트하고자 하는 대상에 대해서만 집중하자

### 결론(Invoice 클래스에서의 금액 계산 방법 변경)  
#### 일반적으로, 변경 지점과 가장 가까운 교차 지점을 선택하는 것이 바람직하다.  
* 고려할 점  
  * 안전성  
    > 변경한 부분에서 가까울 수록 부차적인 복잡도가 줄어든다.  
      >> Ex] 토론 : 말이 장황해지는 걸 막는다.  
  * 테스트 설정  
    > 멀어지면 멀어질수록 테스트 설정 사항이 많아질 수 있다.  
  #### 많은 수의 단계를 거친 후의 기능을 테스트 루틴이 제대로 테스트하는 지 확인하려면
  #### 머리가 컴퓨터처럼 회전해야할 수도 있다.
  
* 결론  
  #### Invoice 클래스의 변경에 대해 getValue 메소드로 테스트하면 어떨까?
  #### 테스트 하네스 내에 Invoice 객체를 생성하고, 테스트 루틴의 동작을 고정시킨 채
  #### 다양한 조건으로 설정을 변경해가며 getValue를 호출할 수 있다.

## 조임 지점 (Pinch Point)

### 상위 수준의 교차 지점  
대부분의 경우, 변경 작업을 위한 가장 좋은 교차 지점은 변경 대상 클래스의 public 메소드일 확률이 높음  ㅇㅈ

#### 하지만, 간혹 public 메소드가 최선의 선택이 아닐 때가 있다.

#### Ex] Invoice 예제의 확장  
* 변경 요구사항 분석  
  * Item 클래스 변경  
    > Invoice 클래스에서 운송 비용의 계산 방법 변경  
    > 운송 업체를 관리하기 위한 필드를 포함하도록 Item 클래스 변경  
  * BillingStatement 클래스 변경  
    > 운송 업체(Item 의 instance variable)별로 구분된 처리가 되도록 변경  
    
* 확장된 청구 시스템에서의 관계도  
  * BillingStatement.makeStatement() ---> Invoice.addItem(item : Item)  
  * Invoice.addItem(item : Item) ---> Item  
  * Item.Constructor(id : int, price : Money)  
   
* 테스트 루틴의 작성 // test fail  
  좀 더 효율적으로 변경 작업을 진행하려면...  
  > 상위 수준의 교차 지점의 존재 여부를 확인  
  * 장점  
    * 의존 관계를 그리 많이 제거하지 않아도 된다.  
    * 코드를 묶음 단위로 취급할 수 있다.
    * **클래스들의 사양을 분명히 드러내는 테스트 루틴**  
      > 좀 더 대규모의 리팩토링 보장  
      
  * BillingStatement의 테스트로부터 접근  
    * BillingStatement의 테스트를 invariant로 이용  
    * Invoice와 Item 클래스의 구조 변경  
    
  * 초기 테스트 루틴  
    > BillingStatement, Invoice, Item을 전부 문서화하는 역할  
    ```
    void testSimpleStatement() {
      Invoice invoice = new Invoice();
      invoice.addItem(new Item(0, new Money(10));
      BillingStatement statement = new BillingStatement();
      statement.addInvoice(invoice);
      assertEquals("", statement.makeStatement());
    }
    ```  
    * 한 개의 Item(품목)을 갖는 Invoice(송장)에 대해 어떤 텍스트를 생성하는 지 확인  
      * 다수의 루틴을 추가함으로써...  
        > Invoice와 Item의 서로 다른 조합에 의해 Invoice가 어떻게 영향을 받는지 확인 가능  
    * 봉합을 도입할 코드 영역을 고려하면서 테스트 케이스 작성 진행  
    
  * 왜 BillingStatement를 교차지점으로 사용했는가?  
    * 클래스들의 **변경에 의한 영향 검출**을 만족시키는 유일한 위치  
    * 가장 쉬운 곳은 아니지만, 적어도 모든 영향을 검출할 수 있는 지점  
    #### 조임 지점 : 영향 스케치에서 영향이 집중되는 곳  
    p254 밑에서 5번째 줄부터 다시하기...

    
