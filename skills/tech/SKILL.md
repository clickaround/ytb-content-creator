---
name: tech-create-video
description: 테크 뉴스 Shorts 영상 생성 (Dailymotion + TTS + 자막). 다국어 지원 (en/ko/ja/es/fr/de/pt/hi).
disable-model-invocation: true
allowed-tools: Bash(curl *)
argument-hint: [language=en] [env=local|prod]
---

# /tech-video

테크 뉴스 기사 기반 YouTube Shorts 영상을 생성한다.

## 파라미터
- `language` — 영상 언어 (기본: en). en/ko/ja/zh/es/fr/de/pt/hi
- `env` — 실행 환경 (기본: local). local/prod
- `title` — 영상 제목 (기본: 테스트용 기본값)

## 실행 단계

### 1. 환경 결정
- `local` → `http://localhost:3124`
- `prod` → `https://YOUR_CLOUD_RUN_URL`

### 2. 페이로드 구성
테스트용 기본 페이로드 (`temp/test-ja-payload.json` 참고):
```json
{
  "workflow_version": "2.3",
  "channel": { "name": "TechTest", "display_name": "Tech Test" },
  "global_config": {
    "language": "{language}",
    "image_generation": "dailymotion",
    "audio": { "voice": "Charon", "tts_provider": "gemini" },
    "video": { "orientation": "portrait" },
    "youtube": { "channelName": "TechTest", "defaultTags": ["AI"], "defaultPrivacyStatus": "private" }
  },
  "videos": [{ "video_id": "tech_test_{timestamp}", "title": "{title}", "scenes": [...] }]
}
```

### 3. API 호출
```bash
curl -s -X POST {BASE_URL}/api/tech/create \
  -H "Content-Type: application/json" \
  -d @payload.json
```

### 4. 상태 추적
```bash
curl -s {BASE_URL}/api/tech/status/{videoId}
```
10초 간격 폴링, `completed` 또는 `error`까지.

### 5. 결과 확인
- local: 파일 직접 열기 (`start "" "{outputPath}"`)
- prod: 다운로드 (`curl -L -o result.mp4 {BASE_URL}/api/tech/download/{videoId}`)

## 관련 파일
- `src/YTB-tech-project/routes.ts` — API 스키마
- `src/YTB-tech-project/services/TechProjectService.ts` — 핵심 로직
- `src/YTB-tech-project/services/video-searcher/DailymotionSearcher.ts` — 영상 클립
- `src/YTB-ffmpeg/utils.ts` — `findFontByLanguage()` 폰트 선택
- `temp/test-ja-payload.json` — 일본어 테스트 페이로드 예시

## 제목 스타일
- 멘탈훈련소 스타일: 흰 글씨 + 두꺼운 검정 외곽선, 배경 없음
- 코드 기본값: `title_bg_color: 'none'`, `title_size: 90`, `borderWidth: 12`

## 다국어 동작
`language` 파라미터 하나로 자동 전환:
- TTS voice (Gemini/Google 언어별 voice)
- TTS 스타일 프롬프트
- Dailymotion 검색 언어
- 폰트 (CJK → NotoSansCJK, Hindi → NotoSansDevanagari, 기타 → BlackHanSans)
- 자막 분할 (CJK → 8글자씩, 라틴 → 단어 그룹)

## 예시
```
/tech-video                          → 영어 테스트 영상 (로컬)
/tech-video language=ja              → 일본어 영상 (로컬)
/tech-video language=ko env=prod     → 한국어 영상 (Cloud Run)
```
