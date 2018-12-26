# 감지와 분리
// 클래스들 간에 의존 관계가 존재하기 때문에 특정 객체들만  
// 테스트 루틴으로 보호하는 것은 매우 어려울 때가 많다.
> 줄줄이 이어지는 의존성 ...  
#### 다른 클래스인 것처럼 위장해서 직접 감지하는 수밖에 없다.

* **테스트 루틴 배치 시 의존 관계를 제거하는 이유**
  * **감지**  
    > 코드 내에서 계산된 값에 접근할 수 없을 때,  
    > 이를 감지하기 위해 의존 관계를 제거한다.  
  * **분리**  
    > 코드를 테스트 하네스 내에 넣어서 실행할 수 없을 때,  
    > 코드를 분리하기 위해 의존 관계를 제거한다.  

## ex] NetworkBridge
```
public class NetworkBridge
{
  public NetworkBridge(EndPoint [] endpoints) {...}
  public void formRouting(String sourceID, String destID) {...}
  ...
}
```
### NetworkBridge
* NetworkBridge는 EndPoint 배열을 입력받는다.  
  > 로컬 하드웨어를 사용해 설정을 관리한다.  
  > 사용자는 NetworkBridge를 통해 트래픽 경로를 제어할 수 있다.  
    >> 하나의 종점에서 다른 종점까지  

* NetworkBridge 클래스는 제어 작업을 수행하기 위해  
  > EndPoint 클래스의 설정을 변경한다.  
  > EndPoint 인스턴스들은 소켓을 열고 특정 장치와 통신할 수 있다.  
  
// 테스트하기 어려울 것 같은 느낌 : **네트워크를 통한 통신**

### 테스트 루틴을 작성해 보자  
* 고려 사항   
  * NetworkBridge instance object는 실제 하드웨어를 자주 호출한다.  
    > 생성을 위해 실제 하드웨어 필요 여부  
  * 하드웨어(네트워크 종점, EndPoint)에 대한 작업 정보  
    > BlackBox 내부를 들여다 보는 것은 불편하다.
    > BlackBox화 한 이유는 있을 것 이다.  

* 다양한 해결책과 최적해  
  * 네트워크상의 패킷을 읽는 코드를 작성해야 하나?  
  * NetworkBridge 인스턴스 생성 시 프리징 현상을 예방하기 위해,  
    새로운 하드웨어가 필요한가?  
  * 네트워크 종점들의 로컬 클러스터를 구성해 테스트하려고,  
    새로운 배선작업을 해야할까?  
> 레이어를, 모듈을, 컴포넌트를 추상화하여 나누는 이유를 고려한다면 ....   

* 최적해에 대한 고민  
  > 우리의 문제는 단순히 NetworkBridge 클래스의 객체를 실행할 수 없으므로  
  > 어떻게 동작할지 직접 확인할 수 없는 것이다.  
    #### 이해하지 못한 것에 대한 과분한 노력과 비용을 투입하려는 건 아닌지 고려해야한다.  
    #### NetworkBridge에 대한 테스트 하네스는 NetworkBridge 클래스의 책임(로직)만 고려한다.  

* **감지와 분리**의 적용  
  일반적으로 둘 다 필요하며, 둘 다 의존관계를 제거하는 이유가 된다.  
  **분리**에 사용되는 기법은 매우 다양하다. (정답이 없는 문제라고 하는)  
  **감지**의 경우에는 일반적인 기법이 존재한다. ((아래)협업 클래스 위장하기)  

## 협업 클래스 위장하기 _ Fake Object(가짜/위장 객체)  
#### 특정 코드에 대한 독립적인 테스트를 위해 다른 코드에 대한 의존 관계 제거가 필요하다.
#### 다른 코드를 별도의 코드(Fake Object)로 대체할 수 있다면 .....

### Fake Object
가짜 객체란 어떤 클래스를 테스트할 때 그 클래스의 **협업 클래스**를 모방하는 객체를 말한다.  

### Ex] POS 시스템  

* POS 시스템 내의 Sale 클래스  
```
public class Sale
{
  public void scan(String barcode)
  {
    1. 고객이 구매중인 품목의 바코드를 읽어 "품목정보"를 얻는다.
      1) 품목정보 : 품목의 이름과 가격
    2. "품목정보"를 금전 등록기의 디스플레이 화면(ArtR56Display)에 출력한다.
  }
}
```

* 화면에 출력하는 코드에 대한 테스트  
  > 코드 내부의 **어느 곳**에서 화면 갱신을 수행하는지 **정확히 구분**할 수만 있다면 쉬워진다.  
  
  * 정확한 구분을 위해 책임을 분리  
    * \[Sale.scan(barcode:String)\]  
      * Sale.scan()의 책임  
        * 바코드를 읽어 "품목정보"를 얻는다.  
        * "품목정보"를 금전등록기의 디스플레이 화면(ArtR56Display)에 출력한다.  
    * \[Sale.scan(barcode:String)\] ---> \[ArtR56Display.showLine(line:String)\]  
      * Sale.scan()의 책임  
        * 바코드를 읽어 "품목정보"를 얻는다.  
        * "품목정보"를 출력한다.
      * ArtR56Display.showLine()의 책임
        * 입력된 정보(line)를 금전등록기의 디스플레이 화면(ArtR56Display)에 출력한다.  
        
  * Fake Object 적용을 위한 추상 계층 추가  
    ```
    public interface Display
    {
     void showLine(String line);
    }
    ```
    * \[Sale.scan()\] ---> \[Display.showLine(line:String)\]
      * Display.showLine()의 책임  
        * 입력된 정보(line)를 디스플레이 화면에 출력한다.  
      * Display의 구현체가 ArtR56Display인 경우  
        * 입력된 정보(line)를 금전등록기의 디스플레이 화면(ArtR56Display)에 출력한다.  
      * Display의 구현체가 FakeDisplay인 경우  
        * 입력된 정보(line)를 우리가 원하는 **가짜 디스플레이 화면**에 출력한다.  
      #### Sale의 특정 코드에 대한 독립적인 테스트를 위해 코드 수정이 필요 없어진다.  
      #### Display를 통한 확장에는 열려있지만 Sale의 특정 코드 수정에는 닫혀있다. // OCP  
      
  * Fake Object를 어떻게 활용(test)할 것인가  
    ```
    public class SaleTest
    {
     public void testDisplayAnItem()
     {
      FakeDisplay display = new FakeDisplay();
      Sale sale = new Sale(display);
      
      sale.scan("1");
      assertEquals("Milk $3.99", display.getLastLine());
     }
    }
    ```
    * 주목해야할 점들  
      * testDiplayAnItem - 테스트명과 책임(테스트 범위)  
        > sale.scan("1")  
          >> 우리가 테스트하고자 하는 것은 바코드로부터 **무엇을** 읽어오는 지가 아니다.  
          >> 우리는 **품목정보 출력**에 대한 테스트를 하고자 한다.  
          >> **품목정보 출력**의 세부 알고리즘이 아닌 **출력 여부**에 대한  
      * FakeDisplay.getLastLine() // Fake Object만의 책임  
        > 독립적이고, 효율적인 테스트  
        ```
        public class FakeDisplay implements Display
        {
         private String lastLine = "";
         public void showLine(String line) { lastLine = line; }
         public String getLastLine() { return lastLine; }
        }
        ```
        > 눈으로 하는 테스트 루틴과 코드로 하는 테스트 루틴  
          >> 테스트를 위한 Fake Object가 어떻게 테스트를 지원하는 지  
          >> lastLine(instance variable) 캡슐화 
          >> getLastLine() 지원  
          
### 가짜 객체(Fake Object)가 진짜 테스트를 지원한다.
실제 기능에 대한 테스트는 아니고,  
Sale 객체가 디스플레이에 어떤 영향(delegating)을 미치는지에 그치지만...
#### 적어도 버그의 원인이 Sale 클래스에 있는지 여부를 빠르게 판단하게 해준다.  
개별 소프트웨어 단위들에 대해 테스트 루틴을 작성하다 보면,
#### 작고 이해하기 쉬운 소프트웨어 단위들이 얻어지게 된다.
이는 우리가 작성한 코드에 대한 합리적인 판단을 내리는 데 도움이 된다.

### 가짜 객체의 특성  
* 가짜 객체의 양면성  
  // 참조 -> 주목해야할 점들 -> FakeDisplay.getLastLine() // Fake Object만의 책임  
  #### Sale 클래스는 가짜 화면을 Display 타입으로 바라보지만
  #### 테스트할 때만큼은 FakeDisplay 타입으로 바라봐야 한다.

* 가짜 객체의 핵심  
  // 참조 -> Ex] POS 시스템  
  객체 지향 언어는 대체로 FakeDisplay 클래스처럼 간단한 클래스를 사용해서 구현한다.  
  * 객체 지향이 아닌 언어의 경우  
    * 대체 함수를 정의  
    * 테스트 시에 접근할 수 있는 값들을 전역 자료구조에 저장  
    * 자세한 설명은 19장 참조  
  
### 모조 객체(Mock Object)  
```
모조 객체는 매우 강력한 도구이고, 다양한 프레임워크가 존재  
모조 객체 프레임워크가 모든 언어에서 사용 가능한 것은 아니다.
```
#### 대부분의 경우 간단한 가짜 객체만으로도 충분하다.
#### 만일 Fake Object를 많이 사용해야 한다면, Mock Object 사용을 고려할 만하다.


```
public class SaleTest
{
 public void testDisplayAnItem() {
  MockDisplay display = new MockDisplay();
  display.setExpectation("showLine", "Milk $3.99");
  Sale sale = new Sale(display);
  sale.scan("1");
  display.verify();
 }
}
```
* Mock Object를 활용한 테스트 루틴  
  * 장점  
    * 예상되는 메소드 호출을 모조 객체에 미리 알려주고,  
      > setExpectation()
    * 실제로 메소드가 호출됐는지 검증하도록 지시할 수 있다.  
      > verify()  
  * verify()  
    **Sale.scan() 내부에서** 설정 값("showLine", "Milk $3.99")대로 작업이 수행됐는지 검증  
    
