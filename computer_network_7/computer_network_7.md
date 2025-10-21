# 7강

**NAT(network address translation, 기능)**

- local(private) network의 모든 기기들은 하나의 IPv4주소를 공유함
- private ip주소는 10/8 - private한 용도로만 사용해야 함
- NAT 장치가 하나의 public ip주소로 온 데이터를 나머지 private주소로 연결된 디바이스들과 연결해줌
- NAT → private 디바이스로 보낼때 어디로 보낼지 어떻게 구분하는가?
    - port넘버를 사용해서 구분함
- 장점
    - ISP가 준 하나의 ip주소로 모든 디바이스가 연결되게 함
    - 로컬 네트워크에서 호스트의 주소를 원하면 막 바꿀 수 있음
    - 로컬 네트워크를 안 건드리고 ISP를 바꿀 수 있음
    - 보안측면에서 이점이 있음, 로컬 네트워크를 가려줌
- 구현법
    - 밖으로 나가는 데이터그램의 소스 ip 주소를 밖에서 쓰는 ip주소, 포트번호로 바꿔줌
        - 그렇게 바꾼걸 NAT translation table에 저장
        - 도착한 패킷은 NAT translation table을 보고 다시 바꿔서 목적지 설정함
- 논란 (NAT는 꼼수다!, 원론주의자들이 싫어함)
    - 라우터는 3계층 너머를 처리하면 안됨 → 나트는 포트번호를 건드림
    - 주소 부족은 IPv6로 해결되어야 함 (NAT를 통해서가 아니라)
    - end-to-end argument(네트워크 내부는 소켓 교환에만 집중)를 깸
    - NAT 내부에서 서버를 열면?
        - 모든 트래픽이 NAT를 통과해야 함 (곤란)
    - 그렇지만 집, 기관, 셀룰러 등에서 잘 쓰는중
        - 특히 IoT에서 잘 사용중

**IPv6 (v5는 테스트)**

- motivation
    - 주소고갈 : 32-bit  IPv4는 주소공간이 고갈됨
        - 근데 IPv6는 IPv4와 호환되지 않음 → IPv4가 문제가 많다고 생각
    - processing/fowarding 빨라짐 : 헤더의 길이를 40byte로 고정 (option X)
    - 각각의 flow에 대해 서로 다른 처리를 할 수 있게 함
- format
    
    ![Untitled](Untitled.png)
    
    - ver : 6
    - pri : prioriy (flow를 위한 개념, 일단 구현만 해둠)
    - next hdr : payload에 들어간 데이터의 성격
    - hop limit : TTL에 해당하는 값, 수명
    - 첵섬, 분해/재조립, option 없음
- transition
    - flag day(기점)가 없음 - 한번에 모든 라우터를 버젼 업그레이드 할 수 없음
    
    Q : 어떻게 두 버젼을 공존 시킬까? (호환이 안되는데?)
    
    A : tunneling(터널링) : 6패킷이 4패킷의 payload(data)로 포함되어 전달됨
    
    → IPv4만 이해하는 라우터 많기 때문에 (셀룰러 같은데서 많이 사용함)
    
    ![Untitled](Untitled%201.png)
    
    - 터널링을 위한 encapsulation (패킷안의 패킷)
    - IPv6 ↔ IPv6 - 이서넷 프레임 안에 v6 패킷이 인캡슐 되어서 보내짐
    - IPv6/IPv4 ↔ IPv6/IPv4 - v4 패킷안에  인캡슐 시켜서 IPv4만 인식하는 IPv4 네트워크(터널)를 지나서 다시 디캡슐 시킴
        
        ![Untitled](Untitled%202.png)
        
    - 인캡슐 할때 / 소스 ip 주소 : 터널의 입구, 목적지 ip 주소 : 터널의 출구
    - 목적지가 자기자신(터널출구)이 된 패킷을 받으면 데이터를 보고 v6 패킷으로 바꿀 수 있음
        
        ![Untitled](Untitled%203.png)
        
- adoption
    - Google : ~30%의 클라이언트 접속이 IPv6 접속임
    - NIST : 미국 정부기관의 1/3이 v6 사용가능함
    - 왜 상용화가 오래 걸릴까? (인터넷의 다른 부분의 변화는 엄청난데)
        - 일반 사람들이 아무런 인식이 없음 (큰 상관 없다는 생각)
        - 오히려 NAT같은 꼼수를 이용

**Generalized Fowarding**

- 기존 : 목적지 - ip주소 매칭 (match plus action)
    - 이걸 일반화 시키기
    - 각각의 라우터는 포워딩 테이블이 있어서 거기에 ip주소를 맞춰보고 포워딩 시켰음
- match plus action 추상화
    - destination-based fowarding : 목적지 IP주소에 기반한 포워딩 →
    - generalized fowarding
        - 여러 헤더필드를 결정하는데 동원하자
        - 많은 액션이 가능함 : drop / copy / modify / log packet
    - Flow table abstraction
        - Flow : 여러 헤더 필드들을 조합해서 결정됨 (link, network, transport 등 여러 계층의 헤더)
        - Flow table 에 Flow를 조합해서 다양한 match 알고리즘을 만들 수 있음
        - generalized fowarding : 간단한 패킷-핸들링 규칙
            - match : 패킷 헤더 필드의 값의 패턴
            - actions : match된 패킷에 따라 패킷을 drop, forward, modify 하거나, 컨트롤러로 보냄
            - priority : 다양한 매치룰에 매치 될때 어떤 매치룰을 우선시 할지 정함
            - counters : #bytes 와 #packets
        - 예시
            
            ![Untitled](Untitled%204.png)
            
    - OpenFlow(표준 프로토콜, API) : flow table entries
        - Match, Action, Stats의 entry 정리
            
            ![Untitled](Untitled%205.png)
            
            - ingress Port - 패킷이 들어온 포트
        - 예시
            
            ![Untitled](Untitled%206.png)
            
            - 목적지-기반 포워딩은 목적지 IP주소만 봄
            - 3번째는 블랙리스트를 정하는 것의 원리
            
            ![Untitled](Untitled%207.png)
            
            - MAC(media access control) - MAC 주소 = 하드웨어 주소
            - 이서넷은 이걸 보고 찾아감
        - 매치 - 액션을 다양한 장치에서 일반화해서 제공함
            - 라우터
                - 매치 : longest destination IP prefix
                - 액션 : 포워드
            - 스위치 (이서넷)
                - 매치 : destination MAC address
                - 액션 : 포워드 or Flood (broadcast, 아무나 받아라)
            - Firewall
                - 매치 : IP주소, 포트 등 다양한 헤더
                - 액션 : 허용, 거절
            - NAT
                - 매치 : IP주소, 포트
                - 액션 : 주소, 포트 재작성
        - 예시 (h5 or h6 → h3, h4) (network-wide behavior를 프로그램 하는 법)
            
            ![Untitled](Untitled%208.png)
            
            - Orchestrated table을 컨트롤러가 만들어서 각각의 라우터에 심음
                - Orchestrated : 테이블끼리 꼬이는 케이스가 없음
        - 요즘은 네트워크 behavior 프로그래밍 위한 P4라는 언어가 있음

**Middleboxes**

- 소스-목적지 사이에 위치한 기존의 역할서 벗어난 새로운 역할을 하는 장치
    
    ex) NAT장치, Firewall
    
- NAT장치 : 집, 셀룰러, 기관
- Firewall (보안장비), IDS(침입 감지 시스템) : 기업, 기관, ISP, 서비스 제공자
- Load balancers : 회사, 서비스 제공자, 데이터 센터, 모바일 넷 …
    - 도착하는 서비스 요청 트래픽을 여러 물리적 머신에 분배 (가중치 따라)
- Application-specific : 서비스 제공자, 기관, CDN (사용자에 가장 가까운 컨텐트 서버로 연결해줌)
- Caches : 서비스 제공자, CDN …
- 초기 : 각 자사 소유의 하드웨어 솔루션
- 최근 : 오픈 API 채택한 일종의 whitebox 하드웨어가 됨
    - 더이상 특정 회사의 특화된 녀석이 아님
    - 프로그래머블한 로컬 액션이 가능해짐
- SDN (software-defined networking, 기술) : private/public cloud 에서 control plane을 어떻게 프로그래밍 할 수있는지 하는 그러한 기술의 통칭
- Open Flow → SDN 구현을 위한 기술의 일종
- network functions virtualizations(NFV) : 범용 하드웨어를 이용해 범용 하드웨어를 네트워크 장치처럼 사용하는 것 (ex. 서버급 컴퓨터를 라우터로 사용)
    - networking, computation, storage …

**IP hourglass(모래시계)**

![Untitled](Untitled%209.png)

- IP는 필수적임 (없이 뭔가를 만들 순 없음)

![Untitled](Untitled%2010.png)

- 요즘은 여러 미들웨어들이 많이 추가되어서 옆구리가 튀어나왔음

**인터넷 설계 원리**

- 목표는 연결성임 → IP를 이용해서
- intelligence 는 end-to-end에 있어야 함 (네트워크에 숨겨져 있기보단)
    - intelligence - 비즈니스 로직
- 세가지 믿음에 근간을 두고 개발
    - 단순한 연결성
    - IP
    - 인텔리전스는 end에 위치

**The end-end argument**

- 복잡한 것은 end system에서 구현되는 것이 맞음
    - 내부는 모든 맥락을 파악하기 힘듦 → 불가능, 비효율
- 예시 (데이터의 신뢰성 보장)
    
    ![Untitled](Untitled%2011.png)
    
    - 어떤 것들은 네트워크 안 또는 말단 에서 기능이 구현될 수 있음 (예시처럼)
    - 아래는 가까운데서 다시 보낼수 있음(효율성), 그러나 부담이 커짐
- 지금은 많이 깨지는 중
    - Firewall, NAT 등의 여러 미들웨어의 등장으로

**인텔리전스 위치의 변화**

![Untitled](Untitled%2012.png)