# 🎁Local LoL CLIENT API 에 대한 이해와 사용법 정리🎁

## 1. 들어가기 전에...

LOL API 사용법에 대해서 파고들어 보니, 저희가 정말 필요로 하는 정보들은 Local Computer의 Local API를 사용해야 한다는 것을 알았습니다. 현재 서비스 중인 여러 LOL 실시간 통계 APP들도 PWA 혹은 Electron 형태의 desktopAPP을 서비스 제공을 위해 사용하고 있었습니다. 먼저 팀원분들께 더 깊게 알아보지 못하고, 기획을 말씀드린 점 죄송합니다. 그래도 좀 찾아보니 서버와 연결할 방법은 없는 게 아니었고, Replay API, LOL chat, 닷지 알림 등을 다른 좋은 API도 쓸 수 있다는 점은 긍정적인 것 같습니다. 
  팀원분들의 이해를 돕기 위해 약소하지만, LOL에서 Local API를 만들고 제공하는 원리와 제공 사이트, 사용법, 저희 프로젝트에 적용할 방안 3가지 정도 소개하겠습니다. 감사합니다. 

## 2. LOL에서 Local API를 제공하는 원리와 사이트 소개  

LOL에서 제공하는 Local API는 두 개가 있습니다. 하나씩 설명 드리겠습니다.

### 2.1 LoL 대기화면에 대한 API를 제공하는 LCU

 첫번쨰로 'LOL 로비 화면' 이라 불리는 (본 게임에 들어가기 전에, 아이템샵, 친구와의 채팅, 게임 시작 버튼을 누르는) LOL Client에 대한 API 입니다. 밑은 예시 입니다.

<img src="C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20240112103543104.png" alt="image-20240112103543104" style="zoom:75%;" align="left"/>

해당 API는 LCU(Lol client Update)라는 이름으로 불립니다. 해당 API의 라이프 사이클은 다음과 같습니다. 

| API가 OPEN 되는 시점                 | API 종료 시점                      |
| ------------------------------------ | ---------------------------------- |
| 컴퓨터 사용자가 LOL 아이콘을 누른다. | 컴퓨터 사용자가 LOL CLIENT를 끈다. |

LCU는 LOL의 아이콘 버튼 (leagueoflegend.ex)가 눌리는지로 활성화가 판단됩니다. 
다음은 LCU의 SWAGGER 입니다. 해당 SWAGGER는 LOL client를 실행하는 중에만 사용 가능합니다. 
https://www.mingweisamuel.com/lcu-schema/tool/

<img src="C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20240112104113400.png" alt="image-20240112104113400" style="zoom:80%;" />

해당 API를 통해서 현재 서비스 중인 자동 룬 설정, 챔피언 선택/ 벤 화면에서 팀원의 챔피언 선택마다 승률을 실시간 통계로 보여주는 로직등을 구현합니다.

### 2.2 LOL 본 게임에 대한 API를 제공하는 LIVE LOL CLIENT

LOL의 본 게임 실시간 데이터, Replay를 제공하는 LIVE LOL CLIENT 입니다. 
https://developer.riotgames.com/docs/lol#game-client-api_live-client-data-api

해당 API에서 팀원 전체의 골드 획득량, 룬 정보, 스펠체크, 킬, 뎃, 어시, 용을 누가 죽였는지, 퍼블, 첫번째 포탑 제거 등 LOL 전반에 대한 모든 API를 얻을 수 있습니다. 해당 API가 https://127.0.0.1:2999 를 가리키고 있는데, 해당 주소는 LOL CLient가 켜져있을 때도 접근이 불가능 하고 오직 LOL 본 게임에 진입 했을 때만 활성화 됩니다. 

| Live LoL Client 시작 시점                  | Live LoL Client 종료 시점 |
| ------------------------------------------ | ------------------------- |
| LOL 대기 화면을 지나서 본 게임에 접속한다. | LOL 본 게임에서 나온다.   |

원리는 다음과 같습니다.
LOL이 본 게임에 들어가면 desktop App이 포트 번호 2999번에 게임의 메타 데이터를 상세히 기록합니다. 그 후 게임이 끝나면 2999번에 적은 내용들을 서버로 전송합니다. 그래서 어제 실행해본 결과 RIOT 서버 API에 matchID를 입력 해서, 실행 중인 게임에 대한 바로 접근은 힘든 것으로 확인되었습니다. 

## 3. 사용법

### 3.1 LCU에 대하여 

LCU에서 친구 소환사 목록을 받아서, 친구 소환사와의 채팅을 뽑아올 수 있는데 이는 WebSocket을 사용했습니다. 어제, (특정 친구와의 마지막 채팅을 긁어오는 것 까지는 성공했는데, 제가 반대로 보낼 수 있는지는 모르겠습니다.) LCU 접근을 위해서는 certificate key가 필요한데, 다행히 npm module 중에, LCU와의 소켓 연결, REST API 연결을 제공하는 플러그인이 존재하였습니다. 밑에 링크 남기겠습니다.
 https://github.com/junlarsen/league-connect
여기 설치와 사용법이 적혀있는데, LCU는 한국 사람들 중에도 사용해서 프로젝트를 진행한 경우가 많아서 해당 코드들 참고하면 될 것 같습니다. 
https://velog.io/@gomiseki/series/%EB%84%88-%EC%8C%A9%EB%B0%B0%EC%A7%80%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C-%EB%8B%B7%EC%A7%80-%EA%B2%BD%EB%B3%B4%EA%B8%B0-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8

### 3.2 LoL LIVE CLIENT에 대하여 

해당 클라이언트는 REST API로 연결이 가능한 것을 확인 했습니다. 깃허브 코드나 유튜브를 보면, addListener를 코드에 장착해서, 특정 이벤트 발생 시 마다, 일을 처리하도록도 하는 것 같은데, 공부가 필요합니다. (코드가 1,2년 전꺼거나, 혹은 유튜브에서 라이브 코딩만 하고 깃 허브는 제공하지 않는 경우가 많더라고요...) 해당 API도 certificate key가 필요한데, 단순하게, axios로 Get 혹은 Post 요청 시, header option에 unAuthorized = false로 두시고 RestAPI하셔도 되고, certificate key를 사이트에서 다운받아서 쓰면 됩니다. 근데 certificate key가 모든 Local에서 같고, 위의 LCU에서 league-connect module 내장 함수로도 긁어올 수 있어서 그렇게 의미 있는 작업은 아닌 것 같습니다. 밑은 axios get 예시입니다. 

```javascript
    updateAllGameData(mainWindow) {
        axios({
            url:'https://127.0.0.1:2999/liveclientdata/allgamedata',
            method:'GET',
            timeout: 3000,
            httpsAgent: new https.Agent({
                rejectUnauthorized: false
            })
        }).then((response) => {
            console.log('success');
            let liveData = response.data;
            liveData.scoreStats  = this.getScoreStats(liveData);
            liveData.playerStats = this.getPlayerStats(liveData);
            mainWindow.send('RiotLiveClientData',liveData);
        }).catch((err) => {
            console.log('error');
            mainWindow.send('RiotLiveClientEnd',true);
        });
    }
```

https://developer.riotgames.com/docs/lol#game-client-api_live-client-data-api
해당 페이지에 쓸 수 있는 API가 나와있습니다. 

https://github.com/ar414-com/pansen-panel/tree/master
위의 깃허브는 게임 진행에 따른 실시간 통계를 간략하게 보여주는 프로젝트 입니다. 해당 코드가 2년 전이어서, 그때 당시와 메타 데이터가 달라진 게 좀 있어서 잘 돌아가진 않지만, 해당 내용 참고해서 크롤링 성공하긴 했습니다. 
![image-20240112121358211](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20240112121358211.png)

## 4. 저희 프로젝트에서 해당 기술 사용에 대하여...

팀원분들과 한번 회의를 하면 좋겠습니다. 
원래는 RIOT 홈페이지 내 API에서 모든 유저의 실시간 데이터를 제공해주는 줄로 알았는데, 특정 유저의 실시간 데이터에 접속하려면, https://localhost:2999 에 접속이 필수적이었습니다. 
그래서 저희와 컨셉이 비슷한 서비스 중인 깃허브 프로젝트들을 찾아본 결과 PWA나 Electron DeskTop APP 혹은 exe 파일로 압축 및 배포해서 다운로드 시 시작프로그램에 등록되도록 하였더라고요.

| ① 닷지경보기! (어이없는 5분 정지 당하지 말자!)               | https://velog.io/@gomiseki/series/%EB%84%88-%EC%8C%A9%EB%B0%B0%EC%A7%80%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C-%EB%8B%B7%EC%A7%80-%EA%B2%BD%EB%B3%B4%EA%B8%B0-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8 |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| ② 실시간 킬 뎃 어시, 용처치, 포탑 처지 확인 깃 허브 프로젝트 | https://github.com/ar414-com/pansen-panel/tree/master        |
| ③ 더블킬마다 알림 띄우기 만드는 라이브 코딩 스트리밍 영상<br />(addListener를 통한 웹소켓 통신, 찾은 거 중에 제일 고기능) | https://www.youtube.com/watch?v=2UzI5EI1UpI                  |
| ④ java 메이븐 프로젝트 LoL Client API 정제 및 add Listener, 롤 클라이언트와 웹소켓 실시간 통신<br />(1,2번은 1초마다 크롤링하는 것이라 직관적인데, 이건 약간 어렵습니다... 처음부터 모두 구현되어 있습니다.) | https://github.com/stirante/lol-client-java-api/blob/master/README.md |

그래서 제가 한번 논의해보고 싶은 것은 

1. 웹서비스 -> localhost 통신 방법을 찾아보고, 백엔드 서버에서 바로 데이터를 가져오는 방안을 모색해본다. 

2. PWA로 만들 때, 어제 검색해본 바로는 react에 offline 상황에서도 app이 돌아가도록 하는 serviceWorker라는 모듈이 있더라고요. 해당 모듈이 로컬 파일 및 로컬 포트 접근이 가능하다고 하는데, 이를 이용하여, 사용자의 로컬 컴퓨터 <-> PWA 통신으로 데이터를 읽어서 Web Server와 REST API하기 입니다. 그거 외에 PWA에서 그냥 local host와 연결 및 통신이 된다고는 하는데, 밑의 stackoverflow 질문에서 해당 내용을 찾았습니다.
   https://stackoverflow.com/questions/63157776/can-a-progressive-web-app-pwa-make-rest-requests-to-the-users-localhost

3. ELCETRON app을 hidden 타입으로 만들어서 (사용자 화면에 안 뜨도록), 시작 프로그램으로 설정하여, 롤 실행 시 OS에서 몰래 돌아가도록 만들고, 우리 Web Server와 통신한다. Electron의 경우 auto-launch라는 모듈 플러그인만 APP 내에서 정의하면 시작 프로그램으로 설정이 가능하다고 합니다. 이를 이용해서, ELectron 앞 단은 만들지 않거나 저희도 간단한 닷지 경보기만 만들어 띄우고, Web과 연결하는 것입니다.

   > 이렇게 되면 APP 설치파일을 무조건 깔아야 하지 않나요? 

   맞습니다.  

![image-20240112123257422](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20240112123257422.png)

하지만 OP.GG나 Discord도 이렇게 윈도우용 앱을 따로 두고 있으니, 친구들한테 더 자세한 실시간 정보를 주고 싶은 사람들은 해당 APP을 깔도록 하면 어떨까 하는 제안을 합니다. 그래서 앱을 깐 사람들은 킬 뎃 어시, 실시간 골드, 용 죽인 시점, 포탑 부순 시점, 친구들과의 챗 등등 자세한 정보를 데스크 톱 앱이 실시간으로 긁어서 web server와 RestAPI를 실시하고, 앱을 깔지 않은 사람들은 게임에 들어간지 얼마나 되었는지, 무슨 챔피언을 하고 있고, 스펠은 뭐 끼고 들어갔는지의 정보만 보여주는 것 입니다. 

4. exe 실행 파일을 만들어서 위의 화면 처럼 배포한 뒤, 컴퓨터 부팅 시 실행되도록 설정 합니다. 해당 파일이 실행되어 :2999 포트로부터 실시간 데이터를 긁어오고, 이벤트 발생 시 마다 서버와 REST API 실시 합니다.  -> 이건 바닐라 JS로도 가능해서 가장 쉬운 방법이긴 한데, 최후의 보루로 놔두면 좋을 것 같습니다.