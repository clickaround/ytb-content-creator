---
name: news-create-video
description: 뉴스 채널 Shorts 영상 생성 (네이버TV 클립 + TTS + 자막). 멀티채널 지원.
disable-model-invocation: true
allowed-tools: Bash(curl *)
argument-hint: [channel=news-channel-1|news-channel-2] [env=local|prod]
---

# /news-video

뉴스 채널 YouTube Shorts 영상을 생성한다. 네이버TV에서 뉴스 클립을 자동 검색+다운로드.

## 파라미터
- `channel` — 채널 이름 (기본: news-channel-1). news-channel-1/news-channel-2
- `env` — 실행 환경 (기본: local). local/prod
- `searchPrefix` — 네이버TV 검색 키워드 (기본: 뉴스)

## 실행 단계

### 1. 환경 결정
- `local` → `http://localhost:3124`
- `prod` → `https://YOUR_CLOUD_RUN_URL`

### 2. 페이로드
n8n에서 생성되는 형식. 테스트 시 `docs/NewsProject/finalNode.json` 참고.

핵심 필드:
```json
{
  "global_config": {
    "channel_type": "{channel}",
    "searchPrefix": "{searchPrefix}",
    "audio": { "voice": "Charon", "tts_provider": "gemini" }
  },
  "videos": [{ ... }]
}
```

### 3. API 호출
```bash
curl -s -X POST {BASE_URL}/api/news/create \
  -H "Content-Type: application/json" \
  -d @payload.json
```

### 4. 상태 추적 + 결과
`/api/news/status/{videoId}` → `/api/news/download/{videoId}`

## 관련 파일
- `src/YTB-news-project/routes.ts` — API
- `src/YTB-news-project/NewsProjectService.ts` — 핵심 로직
- `src/YTB-news-project/natv/NatvVideoSearcher.ts` — 네이버TV 검색
- `docs/NewsProject/finalNode.json` — 테스트 페이로드

## 채널 설정
| channelName | YouTube 채널 | 성향 |
|-------------|-------------|------|
| news-channel-1 | NewsChannel1 | 보수 |
| news-channel-2 | NewsChannel2 | 진보 |

## 예시
```
/news-video                                    → news-channel-1 테스트 (로컬)
/news-video channel=news-channel-2             → news-channel-2 테스트
/news-video channel=news-channel-1 env=prod    → news-channel-1 (Cloud Run)
```
