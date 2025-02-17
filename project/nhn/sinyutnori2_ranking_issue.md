## 신윷놀이2 개발 
신윷놀이2 메인 페이지 개발 및 유지보수를 맡게 되었다. 

## 이슈
운영중 메인 페이지 응답이 느리다는 메시지를 받았다. 메인 페이지에 접속해 보니 페이지가 뜨지 않고 오랜 시간 로딩이 발생하고 있었다. 

## 문제 원인1 - 랭킹 서버가 장애인데, 우리 메인에도 문제가 되지?
원인을 분석해 보니, 메인 페이지를 띄우기 위해서는 상점 리스트, 배너리스트, 랭킹 리스트 등의 정보를 호출하고 있었다. 그중 랭킹 리스트를 가져오는 api 의 응답이 느렸다. 랭킹 정보는 외부 api 호출로 정보를 가져오고 있었고, 응답이 느리면 우리 메인 페이지도 영향을 받아 화면이 뜨지 않는 게 문제였다.


## 해결 
랭킹 서버에 이슈가 있더라도 우리 메인 페이지는 정상 동작을 해야 한다고 생각했다. 우선 급하게 메인 페이지 API 호출 부분을 별도 API로 분리하고 프론트에서 별도로 호출하도록 수정했다. 랭킹 정보를 제외한 부분은 정상 동작하도록 개선했다.

그럼에도 여전히 잔존 이슈가 있었다. 한 메서드에서 판수, 다승, 머니 랭킹 정보를 얻기 위해 랭킹 서버에 총 3번의 api 호출을 하고 있었고 하나가 실패하면 랭킹 정보를 얻어오지 못했다. 그래서 API 호출을 각각 분리하여 성공한 건에 대해서는 랭킹 정보를 노출할 수 있도록 개선 작업을 추후 진행했다.


## 문제 원인2 - 메인 페이지 로딩이 왜 이렇게 오래 걸리지? (에러 페이지라도 빨리 떠야 하는 게 아닌가..?)
랭킹 서버 이슈로 메인 페이지는 에러 페이지가 아닌 굉장히 긴 시간 동안 로딩이 지속되다가 종료된다. restTemplate 설정을 보니 readTimeout 설정이 없어서 랭킹 서버가 응답을 줄 때까지 blocking 되는게 원인이었다. 


## 해결 
restTemplate readTimeout을 설정하여 무한정 대기 상태를 개선하고, 기존에 new RestTemplate으로 사용하던 코드도 같이 수정했다.

이후에도 있을 랭킹 서버 장애를 대비해서 circuit breaker 설정을 추가했다. 장애시 우리 쪽에서는 랭킹 서버에 api 호출을 줄여서 빠르게 화면을 렌더링할 수 있게 되었고, 랭킹 서버는 불필요한 부하를 줄일 수 있도록 추가 개선 작업을 진행했다. 


## 회고 
랭킹 서버의 장애로 인해 내 서비스까지 장애로 이어지는 경험을 했다. 
이 경험을 통해서 외부 API 호출 시, 고려해야 할 점이 많다는 걸 생각하게 되었다. timeout 설정은 제대로 되어있는지, 외부 api 호출시 응답이 오지 않을 때는 어떻게 대처할 건지에 대해 생각하게 되었다. 

또한 레거시 코드도 수정하면서, 신윷놀이2에서 사용하는 코드가 신윷놀이1에서도 보였다. 기존에 정상 동작하고 있으니, 복사 붙여넣기를 한거로 보인다. 
이번 일을 겪으면서 기존 코드가 제대로 동작한다고 해서 그대로 가져다 쓰는 건 위험한 일이라는 걸 다시 깨달았다. 기존 코드를 그대로 재사용하는 걸 경계하게 되었다. 