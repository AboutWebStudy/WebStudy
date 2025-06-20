HLS(HTTP Live Streaming)는 **Apple**이 개발한 **비디오 스트리밍 프로토콜**로, 주로 **인터넷을 통해 오디오 및 비디오 콘텐츠를 실시간 또는 주문형(on-demand)으로 전달**하는 데 사용

---

### **핵심 개념**

- **HTTP 기반**
    
    콘텐츠를 일반 HTTP 서버를 통해 전송하므로 **방화벽, CDN, 캐시 서버**와 잘 호환
    
- **Segment(조각) 방식 전송**
    
    전체 비디오를 작게 분할(보통 2~10초)한 `.ts` 파일로 쪼개어 순차적으로 전송
    
- **M3U8 플레이리스트 사용**
    
    `.m3u8` 형식의 **플레이리스트 파일**을 사용해 어떤 세그먼트를 재생할지 정의
    
- **적응형 비트레이트 스트리밍(ABR)**
    
    네트워크 상태에 따라 자동으로 품질(해상도/비트레이트)을 조정해 **버퍼링을 최소화**
    

---

### **HLS 구성 요소**

1. **Master Playlist (m3u8)**
    
    여러 해상도/비트레이트 버전을 링크한 상위 목록
    
2. **Media Playlist (m3u8)**
    
    특정 품질의 세그먼트 목록 (ex: `segment1.ts`, `segment2.ts`...)
    
3. **Media Segments (.ts)**
    
    실제 오디오/비디오 데이터가 들어 있는 짧은 MPEG-TS 파일들
    

---

### 작동 방식 요약

1. 클라이언트가 `.m3u8` 플레이리스트 요청
2. 플레이리스트에 있는 세그먼트(.ts) 파일들을 순차 다운로드
3. 다운로드한 세그먼트를 재생
4. 네트워크 상태 따라 다른 품질의 플레이리스트로 전환 가능

---

### 어디에 사용되나?

- YouTube, Twitch, Netflix 등 OTT 서비스
- 모바일/웹 브라우저 기반 스트리밍
- 라이브 방송 시스템 (뉴스, 스포츠 등)

---

### 장점

- 범용성: 브라우저와 모바일 기기에서 바로 재생 가능
- HTTP 사용: 별도 스트리밍 서버 필요 없음
- 적응형 스트리밍: 다양한 네트워크 환경에서 안정적 재생

## M3U8

`m3u8`은 **HLS(HTTP Live Streaming)에서 사용되는 플레이리스트 파일 형식**으로, **UTF-8 인코딩된 M3U 파일.** 쉽게 말해, **재생할 미디어 파일(.ts 세그먼트 등)의 목록과 스트리밍 정보를 담고 있는 텍스트 파일**

---

### m3u8 파일이 하는 역할

- 스트리밍할 **미디어 세그먼트(.ts)** 들의 **경로**를 나열
- 각 세그먼트의 **길이, 순서** 정보를 포함
- 경우에 따라 **해상도별 스트림 목록(Master Playlist)** 역할도 함

---

### m3u8의 두 종류

1. **Master Playlist**
    
    여러 화질 버전의 Media Playlist를 포함한 상위 목록
    
    ```
    #EXTM3U
    #EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
    low.m3u8
    #EXT-X-STREAM-INF:BANDWIDTH=1400000,RESOLUTION=1280x720
    mid.m3u8
    #EXT-X-STREAM-INF:BANDWIDTH=2800000,RESOLUTION=1920x1080
    high.m3u8
    
    ```
    
    사용자의 네트워크 상황에 따라 적절한 화질의 Media Playlist를 선택
    
2. **Media Playlist**
    
    실제 비디오 세그먼트 목록을 나열
    
    ```
    #EXTM3U
    #EXT-X-TARGETDURATION:10
    #EXT-X-VERSION:3
    #EXTINF:9.0,
    segment1.ts
    #EXTINF:10.0,
    segment2.ts
    #EXTINF:8.5,
    segment3.ts
    #EXT-X-ENDLIST
    
    ```
    
    위 순서대로 `.ts` 파일을 다운로드하고 재생
    

---

### 주요 태그 설명

| 태그 | 설명 |
| --- | --- |
| `#EXTM3U` | m3u8 파일의 시작을 알림 |
| `#EXTINF:<duration>` | 각 세그먼트의 재생 길이(초) |
| `#EXT-X-STREAM-INF` | 마스터 재생 목록에서 각 해상도 스트림 정의 |
| `#EXT-X-ENDLIST` | 리스트가 종료되었음을 의미 (VOD에서 사용) |
| `#EXT-X-TARGETDURATION` | 세그먼트의 최대 길이 |

---

### m3u8 파일은 어떻게 쓰이나?

- 웹 플레이어에서 `.m3u8` 주소를 불러와 스트리밍 시작
- 라이브 방송은 `.m3u8`이 실시간으로 업데이트됨
- 동영상 다운로드 도구에서 `.m3u8`을 분석해 전체 영상 다운로드 가능

## .HLS에서 `.ts` 파일이란?

HLS 스트리밍에서 `.ts`는 **MPEG-TS (MPEG Transport Stream)** 형식의 **비디오 세그먼트 파일**

- **비디오와 오디오 데이터를 포함한 실시간 전송용 포맷**
- `.ts = Transport Stream`
- 스트리밍을 위해 짧게 나뉜 (예: 5~10초) 미디어 조각들

---

### 예시로 비교

| 파일 종류 | 확장자 | 의미 |
| --- | --- | --- |
| 스트리밍 비디오 세그먼트 | `.ts` | MPEG Transport Stream |
| 웹 프로그래밍 언어 파일 | `.ts` | TypeScript 소스코드 |

---

### 스트리밍에서 .ts 사용 예시

```
#EXTINF:10.0,
video1.ts
#EXTINF:10.0,
video2.ts
#EXTINF:10.0,
video3.ts

```

여기서 `video1.ts`, `video2.ts` 등은 **비디오 데이터 조각**들이고, 순서대로 스트리밍
