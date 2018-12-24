# 소프트웨어 변경

## 소프트웨어 코드를 변경하는 네 가지 이유
* 새로운 기능(feature)의 추가  
* 버그 수정  
* 설계 개선  
* 자원 이용의 최적화  

## 기능 추가와 버그 수정

단순히 회사의 로고 위치를 바꾼다면...  
* 요구사항에 대한 **관점**  
  * 고객 관점  
    현업 담당자는 웹사이트를 보고, 부서 회의를 통해  
    * **로고 위치 변경 및 약간의 기능 추가 요청**
      > 약간의 애니메이션 효과  
  
  * 개발자 관점  
    * 완전히 새로운 기능을 추가하는 작업  
    
  * 조직 관점  
    > 개발자가 실제로는 새로운 작업을 해야 함에도 불구하고  
    > 로고 이동을 단순한 버그 수정 정도로만 여기는 때가 많다.  
    
#### 보는 사람의 관점에 따라 새로운 기능 추가인지 버그 수정인지에 대한 판단은 다를 수 있다.
하지만, 어느 경우든
#### 소프트웨어 코드 및 그 밖의 산출물을 변경해야 한다는 사실에는 변함이 없다.  

### 좀 더 중요한 사실 : 동작 변경(behavioral change)
> **새로운 동작을 추가하는 것**과 **기존의 동작을 변경하는 것** 간의 커다란 차이  
  
* **동작**(behavior)
  * **동작**은 소프트웨어에서 가장 중요한 것이다.  
  * 사용자가 원하는 것은 **동작**이다.  
    * 사용자가 원하는 동작을 추가하면,  
      > 사용자는 소프트웨어를 좋아한다.  
    * 사용자가 원하는 동작을 변경하거나, 삭제하면, 버그를 발생시키면  
      > **사용자들은 우리를 더 이상 신뢰하지 않는다.**  
  
  * 웹 페이지 로고 이동  
    * 새로운 동작의 추가  
      > 화면 오른편에 회사 로고 표시  
    * 제거되는 동작  
      > 화면 왼편에 회사 로고의 사라짐  
      
    * **처음부터 화면 왼편에 로고가 없는 상태에서 오른편에 추가**  
      > 새로운 동작이 추가? 제거되는 동작은? 새롭게 로고가 그려질 자리에 무언가가 존재하고 있었는가?  
      > 동작의 변경? 추가? 둘 다?  
   
#### 프로그래머 관점에서의 기준  
* 기존 코드를 변경해야 한다면?  
  > 동작 변경  
* 새로운 코드를 추가하고, 호출할 뿐이라면?  
  > 동작 추가
  
* 코드 관점
```
public class CDPlayer {
  ...
  public void addTrackListing(Track track){}
  ...
}
```
  * 노래 목록을 교체하는 메소드 추가  
  ```
  public void replaceTrackListing(String name, Track track){}
  ```
  * 어떻게 바라봐야 하나?  
    * 애플리케이션에 새로운 동작을 추가했거나, 기존 동작을 변경했을까?  
      > 아니다. (아무 동작도 변경되지 않았다.)  
  
  * CD 플레이어 UI에 새로운 버튼을 추가한다면?  
    // 사용자는 이 버튼으로 노래 목록을 교체할 수 있다.  
    > replaceTrackListing() 메소드에 동작을 추가  
    > 동시에 약간의 동작 변경도 포함 (UI에 대한 변경)  
      >> 어쩌면, UI가 화면에 표시되는 시간이 매우 조금 느려질지도 모른다.  
      
#### 어느 정도의 동작 변경 없이 동작을 추가하는 것은 거의 불가능 하다.
