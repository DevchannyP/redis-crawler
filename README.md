# 🔴 Redis Crawler – Reddit 기반 AI 영상 자동화 시스템

> Reddit 인기 영상 → Whisper 자막(STT) → TTS 음성 → FFmpeg 편집 → 완전 자동 영상 합성

---

## 🧠 프로젝트 개요

**Redis Crawler**는 Reddit 게시물 수집부터 영상 자막화, 오디오 합성, 최종 영상 출력까지의 전체 흐름을 Java 기반으로 자동화한 시스템입니다.

해당 저장소는 코드 전체를 비공개하며, 이 문서는 시스템 작동 원리를 기술 문서처럼 설명합니다.

---

## ⚙️ 전체 파이프라인 구조

[1] Reddit 게시물 수집
⬇
[2] 분류 및 필터링
⬇
[3] yt-dlp로 영상 다운로드
⬇
[4] Whisper STT → 자막 추출
⬇
[5] 영상 편집 (속도/클립 설정)
⬇
[6] Google TTS → 인트로 음성 생성
⬇
[7] FFmpeg로 영상 + TTS 오디오 합성
⬇
[8] 자막 삽입 (SRT → Burn-in)
⬇
🎉 최종 쇼츠 영상 생성 완료

yaml

---

## 📁 폴더 구조

redis-crawler/
├── crawler/
│ ├── AutomationRunner.java # 전체 흐름 제어
│ ├── VideoMetaSearcher.java # Reddit API 수집기
│ ├── PostClassifier.java # 게시물 주제 분류
│ ├── VideoDownloader.java # yt-dlp 영상 저장
│ ├── VideoEditor.java # FFmpeg 클립 생성
│ ├── TTSGenerator.java # Google TTS 연동
│ ├── VideoComposer.java # 영상 + 오디오 합성
│ ├── SubtitleGenerator.java # Whisper 파이썬 호출
│ ├── SubtitleInserter.java # SRT 자막 삽입
├── utils/
│ ├── WhisperSTT.py # Whisper STT 스크립트
│ └── WebDriverFactory.java # 셀레니움 인스턴스 팩토리
├── output/
│ ├── downloads/ # 원본 영상
│ ├── edited/ # 편집 영상
│ ├── subtitles/ # .srt 자막
│ ├── tts_audio/ # mp3 음성
│ └── final/ # 최종 영상

markdown

---

## 🔍 주요 구성 요소 설명

### 1️⃣ VideoMetaSearcher.java

- **기능**: Reddit OAuth2 인증 후 `/r/all` 또는 키워드 기반 핫 영상 게시물 수집
- **핵심 로직**:
  - `fetchVideoPostsFromSubreddit(String subreddit)`
  - `searchRedditVideos(String keyword)`
- **요소**: 제목, URL, selftext를 Map으로 저장
- **필터 조건**: `secure_media.reddit_video` 존재 여부로 영상 판단

---

### 2️⃣ PostClassifier.java

- **기능**: 게시글을 주제(예: animals, fail, music 등)로 자동 분류
- **원리**: 제목 + 본문에 키워드 포함 여부로 다중 분류 처리
- **출력**: `Map<String, List<Map<String, String>>>` 형태로 분류된 리스트 반환

---

### 3️⃣ VideoDownloader.java

- **기능**: yt-dlp를 외부에서 호출하여 mp4 영상 저장
- **안정성**:
  - 최대 3회 재시도
  - Reddit 외 링크 무시
- **출력**: 날짜+제목 기반 파일명으로 저장  
  예: `20250728_CuteDog_001.mp4`

```bash
yt-dlp -f bestvideo+bestaudio --merge-output-format mp4 -o "경로/파일명.mp4" URL
4️⃣ WhisperSTT + SubtitleGenerator.java
기능: Python Whisper 호출 → mp4에서 SRT 자막 생성

구조:

whisper_transcribe.py input.mp4 output.srt

실행 방식: Java → ProcessBuilder → Python CLI 호출

결과물: subtitles/ 디렉토리에 .srt 저장

5️⃣ VideoEditor.java
기능: 영상 속도 및 구간 조절, 클립 추출

FFmpeg 명령 예시:

bash
ffmpeg -i input.mp4 -vf setpts=0.75*PTS -af atempo=1.25 -ss 00:00:03 -t 00:00:15 output.mp4
편집 기준:

1.25배 속도

앞부분 3초 제거

15초 클립 추출

6️⃣ TTSGenerator.java
기능: Google Text-to-Speech API를 호출하여 mp3 파일 생성

HTTP 요청: /v1/text:synthesize 엔드포인트

음성 설정: ko-KR-Wavenet-A 사용

결과물: tts_audio/{category}_intro.mp3

7️⃣ VideoComposer.java
기능: 영상과 오디오를 하나의 mp4로 합성

명령어 구성:

bash
ffmpeg -i video.mp4 -i voice.mp3 -c:v copy -c:a aac -shortest output.mp4
주의: 오디오는 AAC로 인코딩, 길이 맞춤 옵션 -shortest 사용

8️⃣ SubtitleInserter.java
기능: SRT 자막을 영상에 Burn-in 형태로 삽입

FFmpeg 명령 예시:

bash
ffmpeg -i input.mp4 -vf subtitles=sub.srt output_with_subs.mp4
출력 위치: final/ 디렉토리에 저장됨

🔐 코드 비공개 사유
구분	이유
✅ Reddit API 오용 방지	비공개 API 키 포함 + 트래픽 자동화 위험
✅ Whisper 로컬 호출 구조	실행 환경 의존성 큼 (Python + GPU 등)
✅ Google TTS API 연동	유료 API 키 포함 (비공개 필수)
✅ 저작권 보호	Reddit 영상 자동 수집 후 편집은 재배포 위험
✅ FFmpeg 고급 스크립트	시스템별 실행 성공률 확보를 위해 문서화로만 안내

✅ 실행 환경 요약
구성 요소	필요 조건
Java	17 이상
Python	3.10 이상, whisper, torch, ffmpeg-python 설치 필요
FFmpeg	시스템에 설치되어 있고 환경변수 등록되어야 함
yt-dlp	CLI 실행 가능한 위치에 설치 필요
Google TTS	GCP 프로젝트 + API Key 발급 필수

🧑‍💻 제작
Devchanny (박찬희)

GitHub: https://github.com/DevchannyP

Blog: https://devchannyp-github-io.pages.dev/

🧵 한 줄 요약
“Redis Crawler는 Reddit 콘텐츠를 자막, 음성, 편집을 거쳐 완전 자동화 영상으로 재탄생시키는 Java 기반 AI 영상 생성 시스템입니다. (코드는 보안·정책상 비공개)”

yaml

---

필요하시면 이 README에 붙일 `.gitignore`, `whisper_transcribe.py` 예시 스크립트, 또는 `gh-pages`용 소개 페이지도 제공해드릴 수 있습니다.  
👉 다음 요청: `“whisper_transcribe.py 예시 줘”`, `“gitignore도 포함해줘”`, `“사
