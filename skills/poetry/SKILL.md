---
name: poetry-create-shorts
description: Poetry Shorts 파이프라인 — NotebookLM → Seedance → ElevenLabs → YouTube
disable-model-invocation: true
allowed-tools: Bash(curl *)
---

## 용도
시/문학 작품을 Poetry Shorts 영상으로 자동 제작. NotebookLM 오디오 생성부터 YouTube 업로드까지 7단계 E2E.

## 핵심 파일
- `src/YTB-Poetry/` — Poetry Shorts 전체 모듈
- `src/YTB-books-project/src/services/SlideToVideoNode.ts` — Seedance 영상 생성
- `src/YTB-tts/providers/tts/ElevenLabsTTS.ts` — TTS 음성 합성

## 의존성
- NotebookLM (Playwright 자동화)
- Seedance API (image-to-video)
- ElevenLabs API (TTS)
- GCS (Google Cloud Storage, 중간 파일 저장)
- FFmpeg (영상 합성)

## 파이프라인 (7단계)
1. **NotebookLM** — Playwright로 소스 업로드 + 오디오 생성 자동화
2. **GCS** — 생성된 오디오/이미지를 Cloud Storage에 업로드
3. **Seedance** — 이미지를 image-to-video로 변환 (비주얼 배경)
4. **ElevenLabs TTS** — Sarah voice, 0.5x speed로 나레이션 생성
5. **텍스트 오버레이** — 자막/시 구절 오버레이
6. **BGM** — 배경 음악 믹싱 (카더가든 등)
7. **YouTube** — 자동 업로드

## API / 인터페이스
```typescript
// ElevenLabs TTS
const tts = new ElevenLabsTTS({ voice: 'Sarah', speed: 0.5 });
await tts.synthesize(text): Promise<Buffer>

// Seedance
await generateVideo(imageUrl: string, prompt: string): Promise<string>
```

## 다른 프로젝트에 이식하기
1. `src/YTB-Poetry/` 복사
2. ElevenLabs API 키 설정
3. Seedance API 접근 설정
4. GCS 버킷 + 서비스 계정 설정
5. NotebookLM은 Google 계정 필요 (Playwright 인증)

## 주의사항
- **제목/설명 반드시 영어**: 외국인 대상 채널이므로 한국어 금지
- **Seedance 할루시네이션**: 프롬프트에 텍스트 포함 금지 (깨진 글자 생성)
- **슬라이드 오버레이 비활성화**: SlideToVideo에서 오버레이 OFF 규칙
- **크롭**: 9:16 세로 비율 필수 (1080x1920)
- **폰트**: 영문 전용 폰트 사용, 한국어 폰트 혼용 금지
