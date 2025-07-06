# Telegram Audio/Text Memo Bot Guide

## 개요

이 n8n workflow는 Telegram을 통해 음성 메시지와 텍스트 메시지를 받아서 자동으로 Google Sheets에 메모로 저장하는 봇입니다. 음성 메시지는 OpenAI의 Whisper API를 사용하여 자동으로 텍스트로 변환됩니다.

## 주요 기능

- **텍스트 메시지 저장**: Telegram으로 보낸 텍스트 메시지를 바로 메모로 저장
- **음성 메시지 변환**: 음성 메시지를 자동으로 텍스트로 변환하여 저장
- **자동 메타데이터 추가**: 날짜, 시간, 발신자 정보, 메시지 타입 등을 자동으로 기록
- **Google Sheets 연동**: 모든 메모를 구조화된 형태로 Google Sheets에 저장
- **실시간 피드백**: 처리 결과를 즉시 Telegram으로 알림

## Workflow 구조

### 1. 메시지 수신 및 분류
- **Telegram Trigger**: Telegram 메시지를 실시간으로 수신
- **Route by Message Type**: 메시지 타입(음성/텍스트/기타)을 자동으로 분류

### 2. 음성 메시지 처리 경로
```
음성 메시지 수신 → 파일 다운로드 → OpenAI 음성 변환 → 데이터 준비 → 저장
```
- **Get Audio Message**: Telegram에서 음성 파일을 다운로드
- **Transcribe Audio**: OpenAI Whisper API로 한국어 음성을 텍스트로 변환
- **Prepare Audio Memo**: 변환된 텍스트를 메모 형식으로 준비

### 3. 텍스트 메시지 처리 경로
```
텍스트 메시지 수신 → 데이터 준비 → 저장
```
- **Prepare Text Memo**: 텍스트 메시지를 메모 형식으로 준비

### 4. 데이터 저장 및 알림
- **Format Memo Data**: 메모 데이터를 Google Sheets 형식으로 포맷팅
- **Add to Google Sheets**: 포맷팅된 데이터를 Google Sheets에 저장
- **Send Confirmation**: 성공 시 확인 메시지 전송
- **Handle Processing Errors**: 오류 발생 시 에러 메시지 전송

## 저장되는 데이터 구조

Google Sheets에는 다음과 같은 정보가 저장됩니다:

| 컬럼 | 설명 | 예시 |
|------|------|------|
| date | 메모 작성 날짜 | 2025-01-15 |
| memo | 메모 내용 (음성→텍스트 변환 포함) | "오늘 회의에서 논의된 내용..." |
| note | 발신자 정보 및 메타데이터 | "From: 홍길동 (@hong123) - Source: audio" |
| telegram_user_id | Telegram 사용자 ID | 123456789 |
| timestamp | Unix 타임스탬프 | 1705123456 |
| source_type | 메시지 타입 | audio 또는 text |

## 설정 요구사항

### 1. Telegram Bot 설정
- **Bot Token**: Telegram BotFather에서 발급받은 봇 토큰 필요
- **Webhook**: n8n에서 자동 생성되는 웹훅 URL 설정

### 2. OpenAI API 설정
- **API Key**: OpenAI 계정의 API 키 필요
- **언어 설정**: 한국어(`ko`)로 기본 설정되어 있음
- **음성 변환**: Whisper API 사용

### 3. Google Sheets 설정
- **Service Account**: Google Cloud Platform에서 서비스 계정 생성 필요
- **Spreadsheet ID**: `1Qr8_t-hTIaaI4z0bvCzQKE1vxYG-WphAkZEKxWRrGMw`
- **시트 권한**: 서비스 계정에 편집 권한 부여 필요

## 사용 방법

### 1. 텍스트 메모 저장
Telegram 봇에게 일반 텍스트 메시지를 보내면 자동으로 메모로 저장됩니다.

```
사용자: "내일 오전 10시 회의 참석해야 함"
봇: "Memo added successfully! Content: 내일 오전 10시 회의 참석해야 함 Date: 2025-01-15 Source: Text message"
```

### 2. 음성 메모 저장
Telegram 봇에게 음성 메시지를 보내면 자동으로 텍스트로 변환되어 저장됩니다.

```
사용자: [음성 메시지]
봇: "Memo added successfully! Content: [변환된 텍스트] Date: 2025-01-15 Source: Audio (transcribed)"
```

### 3. 지원하지 않는 메시지 타입
사진, 동영상, 스티커 등은 지원하지 않으며, 다음과 같은 안내 메시지가 표시됩니다:

```
봇: "I can only process text messages or voice messages. Send me: A voice message (I'll transcribe it) or A text message (I'll save it directly). Try again with one of these formats!"
```

## 오류 처리

### 1. 음성 변환 실패
- OpenAI API 오류 시 자동으로 오류 처리 경로로 이동
- 사용자에게 재시도 안내 메시지 전송

### 2. Google Sheets 저장 실패
- 네트워크 오류나 권한 문제 시 오류 메시지 전송
- 실패한 메모는 저장되지 않음

### 3. 일반적인 처리 오류
```
봇: "Sorry, something went wrong while processing your memo. Please try again later. If the problem persists, contact support. For audio messages, ensure the file is under 25MB and in a supported format."
```

## 제한사항

- **파일 크기**: 음성 메시지는 25MB 이하만 지원
- **언어**: 음성 변환은 한국어로 최적화되어 있음
- **메시지 타입**: 텍스트와 음성 메시지만 지원
- **동시 처리**: 한 번에 하나의 메시지만 처리 가능

## 보안 고려사항

- Google Sheets에 저장되는 모든 데이터는 평문으로 저장됨
- Telegram 사용자 ID가 기록되어 추적 가능
- 민감한 정보는 메모로 저장하지 않는 것을 권장

## 확장 가능성

이 workflow는 다음과 같이 확장할 수 있습니다:

- **다국어 지원**: OpenAI 설정에서 언어 옵션 추가
- **키워드 분류**: 메모 내용을 분석하여 자동 카테고리 분류
- **알림 기능**: 특정 키워드 포함 시 추가 알림 발송
- **백업 저장**: 여러 저장소에 동시 저장
- **검색 기능**: 저장된 메모 검색 명령어 추가

## 문제 해결

### Webhook 연결 오류
- n8n에서 workflow를 활성화했는지 확인
- Telegram Bot의 웹훅 URL이 올바르게 설정되었는지 확인

### 음성 변환 오류
- OpenAI API 키가 유효한지 확인
- API 사용량 한도를 초과하지 않았는지 확인

### Google Sheets 저장 오류
- 서비스 계정 권한이 올바르게 설정되었는지 확인
- 스프레드시트 ID가 정확한지 확인