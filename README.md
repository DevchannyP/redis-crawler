# 🔴 Redis Crawler – Reddit 기반 AI 영상 자동화 시스템

> Reddit 인기 영상 → Whisper 자막(STT) → TTS 음성 → FFmpeg 편집 → 완전 자동 영상 합성  
> 🎥 Java 기반 엔드투엔드 자동화 시스템. 코드 비공개. 구조 및 작동 원리만 문서화.

---

## 🧠 프로젝트 개요

**Redis Crawler**는 Reddit 인기 영상 게시물을 수집하고, 해당 영상에 대해  
Whisper 기반 STT 자막 생성 → Google TTS 음성 생성 → FFmpeg 편집까지  
모든 과정을 **완전히 자동화**하는 시스템입니다.

최종적으로 TikTok/Shorts/Reels에 적합한 **AI 기반 쇼츠 콘텐츠**를 자동 생성할 수 있습니다.

---

## 🗂️ 전체 자동화 파이프라인

```
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
```

---

## 1️⃣ Reddit 게시물 수집

- Reddit OAuth2 인증을 통해 `/r/all` 또는 키워드 기반 게시물을 수집합니다.
- 영상이 포함된 게시물만 필터링하여 저장합니다.

📸 예시 이미지:

![게시물 수집 예시](https://raw.githubusercontent.com/DevchannyP/redis-crawler/main/redis-crawler/c1.webp)

```java
List<Map<String, String>> redditPosts = metaSearcher.fetchVideoPostsFromSubreddit("all");
```

---

## 2️⃣ 영상 다운로드

- yt-dlp를 통해 `.mp4` 영상 파일을 고화질로 다운로드합니다.
- 3회 재시도 로직, 날짜+제목 기반 고유 파일명 부여 포함

📸 다운로드 동작 예시:

![영상 다운로드 예시](https://raw.githubusercontent.com/DevchannyP/redis-crawler/main/redis-crawler/c2.webp)

```bash
yt-dlp -f bestvideo+bestaudio --merge-output-format mp4 -o "경로/파일명.mp4" URL
```

---

## 3️⃣ 영상 저장 구조

- 날짜/카테고리별로 자동 폴더 구조 생성
- 후처리 단계별 결과를 디렉토리별로 분리 저장

📸 폴더 구조 예시:

![영상 폴더 구조](https://raw.githubusercontent.com/DevchannyP/redis-crawler/main/redis-crawler/c3.webp)

```
output/
├── downloads/     # 원본 영상
├── edited/        # 속도/구간 조정된 영상
├── subtitles/     # Whisper 자막 (.srt)
├── tts_audio/     # TTS 음성 파일 (.mp3)
└── final/         # 최종 합성 영상 (.mp4)
```

---

## 4️⃣ 영상 클립 예시

- 자동 수집된 영상들이 자막 및 편집을 거쳐 쇼츠용 클립으로 구성됩니다.
- TTS 음성과 자막이 삽입된 최종 영상 출력까지 자동화됩니다.

📸 영상 수집/처리 결과:

![자동 수집된 영상 예시](https://raw.githubusercontent.com/DevchannyP/redis-crawler/main/redis-crawler/c4.webp)

---

## ⚙️ 핵심 처리 구성

| 단계       | 클래스 / 도구                             | 설명                                 |
|------------|--------------------------------------------|--------------------------------------|
| 게시물 수집 | `VideoMetaSearcher.java`                   | Reddit API 인증 → JSON 파싱         |
| 분류       | `PostClassifier.java`                      | 제목/본문 키워드 기반 주제 분류     |
| 다운로드   | `VideoDownloader.java` + `yt-dlp`          | CLI 호출 + 재시도 로직 포함         |
| STT 자막   | `WhisperSTT.py` + `SubtitleGenerator.java` | Whisper 로컬 실행, SRT 추출         |
| 영상 편집  | `VideoEditor.java` + `FFmpeg`              | 속도 조정, 구간 클립 생성           |
| TTS 생성   | `TTSGenerator.java`                        | Google TTS API로 mp3 생성            |
| 오디오 합성| `VideoComposer.java`                       | 영상과 TTS 음성 병합                |
| 자막 삽입  | `SubtitleInserter.java`                    | FFmpeg로 자막 Burn-in 삽입           |

---

## 🔐 코드 비공개 사유

| 항목                 | 이유                                                      |
|----------------------|-----------------------------------------------------------|
| Reddit API 인증      | OAuth2 client ID/secret 포함, abuse 방지 목적             |
| Whisper 로직         | 실행 환경(GPU/Python) 의존 → 공개 시 실행 오류 우려      |
| Google TTS API       | 유료 API 키 포함, 보안상 공개 불가                       |
| FFmpeg 스크립트      | 고급 필터/속도 처리 등 시스템별 동작 차이 존재           |
| 저작권 보호          | Reddit 영상 자동 가공 → 재배포 위험 요소 있음             |

---

## ⚙️ 실행 전 요구사항 요약

| 구성 요소 | 필수 조건                                               |
|-----------|----------------------------------------------------------|
| Java      | 17 이상                                                  |
| Python    | 3.10+, `whisper`, `torch`, `ffmpeg-python`               |
| yt-dlp    | CLI 실행 경로에 설치되어 있어야 함                      |
| FFmpeg    | 시스템 PATH 등록 필수                                   |
| GCP TTS   | API Key 및 인증 JSON 필요 (Google Cloud Console 설정)   |

---

## 🙋🏻‍♂️ 제작

- **Devchanny (박찬희)**  
- GitHub: [https://github.com/DevchannyP](https://github.com/DevchannyP)  
- Blog: [https://devchannyp-github-io.pages.dev/](https://devchannyp-github-io.pages.dev/)

---

## 🧵 한 줄 요약

> Redis Crawler는 Reddit 콘텐츠를 기반으로 자막 생성(STT), 오디오 생성(TTS), 영상 편집(FFmpeg)을 수행하는  
> Java 기반의 완전 자동 영상 생성 시스템입니다.  
> **실제 코드는 보안상 비공개**이며, 본 문서는 시스템의 원리와 구조를 설명합니다.
