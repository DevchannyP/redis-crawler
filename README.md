# 🔴 Redis Crawler – Reddit 기반 AI 영상 자동화 시스템

> Reddit 인기 영상 → Whisper 자막(STT) → TTS 음성 → FFmpeg 편집 → 완전 자동 영상 합성  
> 🎥 Java 기반 엔드투엔드 자동화. 코드 비공개. 구조 및 작동 원리만 문서화.

---

## 🧠 프로젝트 개요

**Redis Crawler**는 Reddit 게시물을 수집하고, 해당 게시물의 영상을 자동으로 다운로드 → 자막화(STT) → 편집 → 음성(TTS) 합성을 거쳐  
최종적으로 **쇼츠 형태의 자동 영상 콘텐츠**를 생성하는 시스템입니다.

---

## 🗂️ 전체 자동화 흐름도

[1] Reddit 게시물 수집
⬇
[2] 주제 분류 및 필터링
⬇
[3] yt-dlp로 영상 다운로드
⬇
[4] Whisper로 자막 추출
⬇
[5] 영상 클립 편집 (속도, 구간)
⬇
[6] Google TTS 음성 생성
⬇
[7] 영상과 오디오 합성
⬇
[8] 자막 삽입
⬇
🎬 최종 영상 자동 생성


---

## 1️⃣ Reddit 게시물 수집

게시물은 Reddit의 `/r/all`, 특정 키워드, 혹은 `trending` 기준으로 Reddit OAuth2 인증 후 API를 통해 수집됩니다.

📸 수집된 게시물의 실제 예시:

![게시물 수집 예시](./redis-crawler/c1.webp)

- 각 게시물에는 `title`, `url`, `selftext`, `permalink` 정보가 포함됨
- 영상이 포함된 게시물(`reddit_video`가 존재하는 경우)만 필터링

```java
List<Map<String, String>> redditPosts = metaSearcher.fetchVideoPostsFromSubreddit("all");

2️⃣ 영상 다운로드
유효한 Reddit 게시물 URL에서 yt-dlp를 사용하여 고화질 .mp4 파일로 영상 다운로드를 수행합니다.
중복 방지를 위해 날짜 기반 파일명, 재시도 로직, 비정상 종료 처리까지 포함합니다.

📸 다운로드 UI / 명령어 처리 흐름:


yt-dlp -f bestvideo+bestaudio --merge-output-format mp4 -o "경로/파일명.mp4" URL


3️⃣ 영상 저장 구조
다운로드된 영상과 처리 중간 결과물은 날짜별로 자동 정리됩니다.
예: downloads/20250728/, edited/20250728/, final/20250728/


output/
├── downloads/     # 원본 영상
├── edited/        # 속도/구간 조정된 영상
├── subtitles/     # Whisper 자막 (.srt)
├── tts_audio/     # TTS 음성 파일 (.mp3)
└── final/         # 최종 합성 영상 (.mp4)



4️⃣ 영상 클립 예시
여러 Reddit 영상이 자동으로 수집, 다운로드되어 다음처럼 클립이 구성됩니다:

📸 자동 수집된 영상 모음 미리보기:


각 영상은 Whisper 자막 → 편집 → 오디오 삽입을 거쳐 최종본으로 생성됨

모든 과정은 Java + FFmpeg + Python(Prompt 기반)으로 비동기 처리

⚙️ 핵심 처리 구성
단계	클래스 / 도구	설명
게시물 수집	VideoMetaSearcher.java	Reddit API 인증 → JSON 파싱
분류	PostClassifier.java	키워드 기반 다중 분류
다운로드	VideoDownloader.java + yt-dlp	CLI 프로세스 실행 + 파일명 정리
STT	WhisperSTT.py + SubtitleGenerator.java	Python 스크립트 호출 방식
영상 편집	VideoEditor.java + FFmpeg	구간 추출 + 속도 조정
TTS 생성	TTSGenerator.java	Google TTS API 호출 후 .mp3 저장
오디오 합성	VideoComposer.java	영상 + 오디오 + 시간 조정
자막 삽입	SubtitleInserter.java	.srt 파일을 Burn-in 방식 삽입

🔐 코드 비공개 사유
항목	이유
Reddit API 키 포함	OAuth2 인증 및 abuse 방지
Google TTS API 사용	유료 서비스, 키 유출 위험
Whisper 구조	실행 환경(GPU/Python)에 의존
저작권 이슈	Reddit 원본 영상 자동 가공은 재배포 위험
FFmpeg 고급 커맨드 사용	시스템 환경별 실행 결과가 다름

⚙️ 실행 전 요구사항 요약
구성 요소	필수 조건
Java	17 이상
Python	3.10+, whisper, torch, ffmpeg-python
yt-dlp	CLI 실행 가능한 경로에 설치
FFmpeg	PATH 등록 필수
GCP TTS	API Key 및 JSON 인증 설정 필요

🙋🏻‍♂️ 제작
Devchanny (박찬희)

GitHub: https://github.com/DevchannyP

Blog: https://devchannyp-github-io.pages.dev/

🧵 한 줄 요약
“Redis Crawler는 Reddit 콘텐츠를 기반으로 자막, 오디오, 편집까지 완전 자동 영상 생성이 가능한 Java 기반 AI 자동화 시스템입니다. 코드는 보안상 비공개입니다.”
