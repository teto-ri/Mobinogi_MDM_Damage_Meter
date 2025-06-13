# 마비노기 모바일 전투 분석 및 시각화

## 소개

![image](https://github.com/user-attachments/assets/52f894e4-95d7-4929-9f9d-48c23be37148)
![image](https://github.com/user-attachments/assets/b5c09f62-662b-42ed-afb7-52e627b1cb7a)


본 프로젝트는 커뮤니티에서 공개된 원본 데미지 로그 분석기(https://github.com/MabinogiMeter/MDM/releases/tag/v1.0.0), (https://github.com/pblyat/MabinogiTools/tree/main/DamageMeter) 를 기반으로

MDM_origin_base는 비공식 로그 분석기를 프록시 중계로 확장해, 원본 분석기에 편리하게 분석 가능한 인터페이스를 추가한 버전입니다.

Decomplie_new_base는 위 분석기들을 참고하여 디컴파일 및 실행가능 코드로 복원하여 리눅스, 도커 기반으로 실행가능하게 인터페이스를 추가한 버전입니다.

디컴파일 및 구조 이해에는(https://arca.live/u/@absolute0) 의 도움을 받았습니다.

## 목적

마비노기 모바일을 플레이하며 파티에서 “누가 뭘 잘하고 있는지” 알 수 없어서 답답했고, 자동전투 유저와 수동 유저의 차이를 체감하지만 수치로 확인할 수 없었습니다. 

커뮤니티에서 제공된 비공식 로그 분석기를 참고해 패킷 구조를 이해하고, 
실시간 시각화 도구와 분석 지표를 구현했고, 플레이 스타일 비교/분석에 사용하였습니다.

본 프로젝트에선 직접적인 패킷 분석을 다루지 않으며 결과 데이터 분석 및 시각화UI개선에 초점을 둡니다.

해당 데이터는 공개된 도구에 의해 노출되는 수준이며, 게임 플레이에 영향을 주거나 데이터 변조, 유출을 포함하지 않습니다.

또한, 개인 연구 목적으로 진행되었음을 밝힙니다.

> ** 2025.06.11 21:30 자로 마비노기 모바일에서 비인가 프로그램 사용 자제 공지가 있었습니다. 

> [공지 링크](https://mabinogimobile.nexon.com/News/Notice/2946578) 참고 바라며, 연구 및 스터디용으로만 살펴보고, 해당 프로그램 사용은 자제해주시길 바랍니다.

## 기능

- MDM_origin_base

  원본 MDM툴은 소스가 비공개이며 localhost에서만 데이터를 확인 가능했으나, 

  본 프로젝트를 이용해서 VM이나 포트미러링이 가능한 기기에서 MDM패킷 분석기를 실행하고

  이를 같은 네트워크 안의 모바일/웹에서 필요한 실시간 전투 데이터를 시각화해, 유저별 기여도, 자동화 패턴, 스킬 사용 주기 등을 분석할 수 있습니다.

- Decomplie_new_base

  최대한 MDM미터기의 출력과 유사하게 복원한 버전입니다. MDM툴과 약간의 패킷 해석 차이가 있으며,

  스킬 해석 방법 중 스킬 이펙트 표시를 위해 미리 도착한 패킷으로 시전에 시간이 걸리는 스킬 분석의 탐지율을 올린 것이 특징입니다.  

  출력은 위와 동일하되, 여러 속성 데미지 플래그가 추가되었으며 위와 같은 시각화 기능을 수행하며 더욱 풍부한 정보를 바탕으로 다양한 분석 기능을 추가하였습니다.

  자동 전투 여부 판단은 단순 반복 여부가 아닌, 시퀀스 구조 + 간격 정규성을 조합하여 판단했습니다.

    - 시퀀스 기반 탐지
    전투 로그를 순차적으로 분석하여, ‘스킬–평타–스킬–평타’ 등의 고정된 루프 구조가 반복되는지를 확인했습니다.

    - 간격 기반 탐지
    각 유저별 스킬 사용 간격의 평균과 표준편차를 구해, **간격이 일정(σ < 일정 기준)**하고 주기가 너무 짧은 경우 자동 전투로 간주했습니다.

    결과적으로, 30초 내 일정 간격으로 3회 이상 같은 시퀀스를 반복하고, σ가 0.4초 이하이며, Dps가 상대적으로 낮은 유저를 자동 전투로 탐지합니다.

## 가상화 테스트 환경

필자는 윈도우의 기본 가상시스템인 Hyper-V를 이용하여 포트미러링 환경을 구성했습니다.

1. 윈도우10 VM 설치 및 실행 참조 링크 (https://blog.naver.com/yujuit/223237345171)

2. 윈도우 환경의 VM 세팅이 끝났다면, 호스트의 패킷이 분석기에 들어오는지 확인합니다.
   
   2-1. VMware의 경우, 네트워크 설정을 브릿지 모드로 실행하면 됩니다.

   2-2. Wsl의 경우, wsl2를 이용하고 네트워크 어댑터를 win11부터 지원되는 mirrored 모드로 설정해야합니다.

   2-3. Docker의 경우, network를 host로 실행해야합니다.

   2-3. Hyper-V의 경우 호스트PC -> VM 안으로 패킷을 전달하도록 수동 설정이 필요합니다.

      먼저 Hyper-V 관리자 -> 가상 스위치 관리자 -> 새 가상 네트워크 스위치 -> 외부, 만들기로 네트워크 스위치를 만들어줍니다.
   
      다음, 호스트 PC에서 Powershell 에서 다음과 같은 명령어를 수행해주세요.
   
      ### 1. 가상 스위치에 분석용 포트 추가
      Add-VMNetworkAdapter -VMName "<VM이름>" -Name "Monitor" -SwitchName "<가상 스위치 이름>"
      
      ### 2. Monitor 어댑터를 미러링 대상(Destination)으로 설정
      Set-VMNetworkAdapter -VMName "<VM이름>" -Name "Monitor" -PortMirroring Destination
      
      ### 3. 외부 포트를 미러링 원본(Source)으로 설정
      $ExtPortFeature = Get-VMSystemSwitchExtensionPortFeature -FeatureName "Ethernet Switch Port Security Settings"
      $ExtPortFeature.SettingData.MonitorMode = 2
      Add-VMSwitchExtensionPortFeature -ExternalPort -SwitchName "<가상 스위치 이름>" -VMSwitchExtensionFeature $ExtPortFeature
      
      ### 4. Microsoft NDIS Capture 확장 활성화
      Enable-VMSwitchExtension -VMSwitchName "<가상 스위치 이름>" -Name "Microsoft NDIS Capture"

5. VM에 패킷 캡처용 winPcap(https://www.winpcap.org/install/bin/WinPcap_4_1_3.exe) 또는 nPcap(https://npcap.com/dist/npcap-1.82.exe)을 설치합니다.

6. MDM툴 버전의 경우,
  6-1. run.bat 실행 후, http://{vm_ip}:8000 으로 접속합니다.

7. Decomplie버전의 경우,
   7-1. mobi_capture.py or mobi_capture.exe 실행 후 mobi_meter.html 실행


## 비공식 패킷 분석기 구조

>  본 내용은 공개된 분석기를 디컴파일, 리버싱하여 이해한 동작 구조입니다.  
---

### 🔄 데이터 처리 절차 (구조 이해 요약)

- **1. TCP 패킷 수신**
  - Scapy의 `AsyncSniffer`를 통해 **포트 16000**에서 송신되는 TCP 패킷만 감지
  - Raw 데이터가 포함된 패킷만 수집하여 `Queue`로 전달

- **2. 세그먼트 조립**
  - TCP는 데이터를 조각내어 전송하므로 **시퀀스 기반으로 순서 정렬**
  - 세그먼트 누락을 고려하여 **완전한 블록이 수신될 때까지 버퍼에 누적**
  - **패킷 시작/종료 시그니처**를 기준으로 하나의 데이터 블록 구성  
    - 시작: `3A 04 00 00 00 00 00 00`  
    - 종료: `04 00 00 00 00 00 00 00`

- **3. 패킷 필터링**
  - 헤더가 유효하지 않거나 의미 없는 데이터 (`\x00...`, 특정 매직넘버 등)는 제외
  - 내부에 `0x03050000`(데미지 타입 시그니처) 포함 여부를 기준으로 유효성 확인

- **4. 압축 해제**
  - 압축 알고리즘 종류의 경우 'Brute Force'로 해제
  - 내부 데이터가 `Brotli`로 압축된 경우 자동으로 해제
  - 이후 바이너리 형식의 상세 정보 추출

- **5. 전투 정보 추출**
  - 다음과 같은 구조로 순차적으로 추출:
    - **공격자 ID (8바이트)**: 누가 공격했는지
    - **피격자 ID (8바이트)**: 누가 맞았는지
    - **스킬 이름 길이 (4바이트)** → **스킬 이름 (가변 길이)**: 사용된 기술 이름
    - **데미지 수치 (4바이트)**: 실제 피해량 (특정 값이면 무효)
    - **부가 정보 (12바이트)**: 사용되지 않지만 그대로 저장됨
    - **속성 플래그 (20바이트)**: 크리티컬, 속성(불/얼음/전기 등), 연타 등 효과 정보
    - **스킬 ID (8바이트)**: 고유 기술 ID

- **6. 정보 후처리**
  - 스킬명이 `Idle`인 경우, 직전에 수신된 `Skill` 정보를 시퀀스/시간 기준으로 매칭하여 보정
  - 스킬명이 없는 경우 `DOT_FIRE`, `DOT_ICE` 등 속성 기반 이름을 자동 생성
  - 플래그는 각 바이트 비트를 읽어 True/False로 변환
    - 예: `crit_flag=True`, `fire_flag=False`, `multi_attack_flag=True` 등
  
  각 플래그는 바이트 배열의 특정 위치에서 **비트 마스크**로 추출

  ```python
  FLAG_BITS = (
      (0, 'crit_flag', 0x01),
      (0, 'unguarded_flag', 0x04),
      (3, 'fire_flag', 0x40),
      (4, 'ice_flag', 0x01),
      ...
  )
  ```

- **7. 로그 변환 및 전송**
  - 추출한 정보를 `|` 구분자로 나열해 문자열로 만듦
  - WebSocket으로 외부 클라이언트(시각화 도구)로 실시간 전송

---

패킷 내부의 전투 관련 데이터는 보통 다음 순서로 구성됨.

```
[스킬 이름 (유니코드)] + [데미지 (4바이트 정수)]
```

- **스킬 이름**은 UTF-16LE 형식 (한 글자당 2바이트)으로 저장됨
- **데미지**는 4바이트 리틀엔디언 정수로 저장됨

---

### 패킷 예시

#### 원시 데이터 (16진수)

```
41 00 72 00 62 00 61 00 6c 00 69 00 73 00 74 00 
5f 00 47 00 75 00 73 00 74 00 69 00 6e 00 67 00 
42 00 6f 00 6c 00 74 00 5f 00 30 00 31 00 
e1 09 00 00
```

#### 구성 해석

| 항목         | 내용                                                              |
|--------------|-------------------------------------------------------------------|
| 스킬 이름    | `Arbalist_GustingBolt_01` (총 44바이트, UTF-16LE로 인코딩됨)        |
| 데미지 값    | `e1 09 00 00` → 0x09E1 (리틀엔디언) = **2529**                      |

#### 최종 해석 결과

```json
{
  "skill_name": "Arbalist_GustingBolt_01",
  "damage": 2529
}
```

---

### 참고

- UTF-16LE은 문자를 2바이트씩 표현하기 때문에, A(0x41)는 `41 00`으로 저장
- 데미지는 항상 4바이트이며, 리틀엔디언 방식으로 **뒤에서부터 읽습니다**:
  - `e1 09 00 00` → `0x000009e1` = 2529

---

### ✅ 분석기 출력 형식

전투 데이터는 다음과 같이 `|` 구분자로 구분된 문자열 형태로 출력

```
timestamp | used_by | target | skill_name | skill_id | damage | crit_flag | addhit_flag | unguarded_flag | break_flag | first_hit_flag | default_attack_flag | multi_attack_flag | power_flag | fast_flag | dot_flag | ice_flag | fire_flag | electric_flag | holy_flag | bleed_flag | poison_flag | mind_flag
```

총 23개의 항목 로그데이터

---

### 실제 출력 예시

분석기가 출력하는 예시 로그

```
1717821123456|fa1a32c41b9d4e5f|dd4b21e94e8a6a33|Arbalist_GustingBolt_01|0011223344556677|2529|1|0|0|0|0|1|0|0|1|0|0|1|0|0|0|0|0
```

#### 항목별 의미

| 항목 이름               | 예시 값                      | 설명 |
|------------------------|------------------------------|------|
| timestamp              | 1717821123456                | 로그 생성 시각 (ms 단위, epoch 기준) |
| used_by                | fa1a32c41b9d4e5f             | 공격자(캐릭터)의 ID (8바이트 hex) |
| target                 | dd4b21e94e8a6a33             | 피격 대상의 ID (8바이트 hex) |
| skill_name             | Arbalist_GustingBolt_01      | 사용한 스킬 이름 |
| skill_id               | 0011223344556677             | 스킬의 고유 식별자 |
| damage                 | 2529                         | 데미지 수치 |
| crit_flag              | 1                            | 크리티컬 여부 (1=Yes, 0=No) |
| addhit_flag            | 0                            | 추가 타격 여부 |
| unguarded_flag         | 0                            | 방어 무시 여부 |
| break_flag             | 0                            | 가드 브레이크 여부 |
| first_hit_flag         | 0                            | 선공 여부 |
| default_attack_flag    | 1                            | 일반 공격 여부 |
| multi_attack_flag      | 0                            | 멀티 타격 여부 |
| power_flag             | 0                            | 강타 여부 |
| fast_flag              | 1                            | 연타 여부 |
| dot_flag               | 0                            | 도트 데미지 여부 |
| ice_flag               | 0                            | 빙결 속성 여부 |
| fire_flag              | 1                            | 화염 속성 여부 |
| electric_flag          | 0                            | 전격 속성 여부 |
| holy_flag              | 0                            | 신성 여부 |
| bleed_flag             | 0                            | 출혈 여부 |
| poison_flag            | 0                            | 중독 여부 |
| mind_flag              | 0                            | 정신 공격 여부 |

---

## 🛠️ 의존성

- `scapy`
- `websockets`
- `brotli`

### 주의
- 단순 분석 도구라도 **게임사 정책에 따라 제재**될 수 있으며, 본 코드는 **학습 목적의 구조 분석 참고용**입니다.
- 게임사의 **이용약관을 위반할 수 있는 환경**에서는 사용을 자제해주시길 바랍니다.

---

## 🖥️ 웹 기반 실시간 전투 시각화 도구

해당 프론트엔드는 툴에서 WebSocket으로 전송되는 전투 로그를 실시간 수신하여, 시각적이고 직관적인 방식으로 **전투 데이터를 분석, 정리, 공유**할 수 있도록 설계되었습니다.

---

![image](https://github.com/user-attachments/assets/76855de4-c77f-48c4-bf98-05613c79275e)


### 📌 주요 기능 요약

| 기능                        | 설명                                                                 |
|-----------------------------|----------------------------------------------------------------------|
| 전투 로그 실시간 수신       | WebSocket을 통해 데미지 로그 실시간 수신                |
| 플레이어 DPS 순위 시각화     | 클래스별 색상과 기여도 바 차트 기반으로 총 데미지, DPS, 기여도 비율 출력      |
| 스킬 상세 분석              | 각 유저의 스킬별 데미지, 타수, 평균/최대/최소 데미지, 상태 이상 확률 등 정리   |
| 누적 데미지 변화 그래프      | 시간 경과에 따른 각 유저의 누적 데미지 라인 그래프 제공                      |
| 자동전투 탐지               | 스킬 사용 간격이 일정한 경우 자동전투 패턴 감지 (평균/표준편차 기반 분석)      |
| 전투 기록 저장/재생         | 최대 20개의 전투 기록 저장 후 언제든 불러와서 리플레이 가능                   |
| PNG 캡처 기능               | 전체 전투 개요 또는 스킬 상세 포함화면을 캡처하여 이미지 저장 가능              |
| 사이드바 토글 기능          | 전투 기록 창을 버튼으로 열고 닫을 수 있어 UI 집중도 향상                        |

---

### 🧱 화면 구성

#### 1. 전투 개요 정보

- 전투 시간 (초 단위)
- 총 누적 데미지
- 공대 RDPS (클래스 감지된 유저만 포함)
- 참여 유저 수

#### 2. 플레이어 순위

- 플레이어 별 총 데미지 및 DPS
- 클래스별 색상으로 시각적 구분
- 클릭 시 스킬 상세 정보 펼쳐보기 가능
- 각 스킬의 타수, 크리율, 추가타율, 평균/최소/최대 데미지, 상태이상 비율 등 표시

#### 3. 전투 데미지 그래프

- 시간(x축) 대비 누적 데미지(y축) 변화 그래프
- 클래스별 고정 색상 라인 제공
- 전투 종료 후 리플레이 모드에서도 사용 가능

#### 4. 전투 기록 저장/불러오기

- 자동/수동 종료 시 로컬 스토리지에 기록 저장
- 최대 20개 저장되며 오래된 기록은 자동 삭제
- 사이드바에서 기록 선택 → 리플레이 가능

#### 5. PNG 캡처 기능

- 개요만 캡처 / 스킬 상세까지 포함 캡처 두 가지 버튼 제공
- html-to-image 기반 PNG 이미지 자동 다운로드

---

### ⚙️ 기타 동작

- `Idle` 스킬명은 직전 스킬과 시퀀스/시간 매칭으로 보정
- 클래스 감지는 스킬명 내 포함 키워드 기준 (`Arbalist`, `FireMage` 등)
- 상태이상 플래그(`dot_flag`, `fire_flag` 등)를 통해 `DOT_ICE`, `DOT_FIRE` 등의 기술 자동 생성
- 전투 간 자동 감지: 10초 이상 로그 수신이 없으면 자동 종료
- 전투 중 화면은 1초마다 갱신되며, 연산이 필요한 항목은 5초마다 동작
- 자동전투 유저는 최근 30초간의 로그를 기반으로 탐지됨.
---

### 🛠️ 의존 라이브러리

- Chart.js (`canvas` 기반 라인 그래프)
- html-to-image (`캡처 기능`)
- WebSocket API
- Vanilla JS (jQuery 미사용)

---

### 🧪 테스트 환경

- 브라우저: Chrome 기반 최신 브라우저 권장
- 서버: Python WebSocket + Scapy 패킷 분석기 필요

> 💡 개발자 옵션에서 `logRawPacket`, `console.log(...)` 등을 통해 디버깅 가능

---

### 🎯 전체 연동 흐름

```
┌──────────────┐        WebSocket         ┌────────────────────────────┐
│  Scapy 기반   │ ───────────────────────▶ │  웹 클라이언트 (viewer.html) │
│    분석기     │                         └────────────────────────────┘
└──────────────┘     JSON 포맷 로그 전송
```

---

### 📝 참고

- 모든 연산은 클라이언트에서 수행되며, 별도 서버 저장소 없음
- 수신된 데이터는 최대 30초간 저장됨 (`rawLogsPerUser`)
