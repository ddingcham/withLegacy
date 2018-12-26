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

