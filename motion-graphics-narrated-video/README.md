# motion-graphics-narrated-video

한국어 원고(성경 여정, 서신서 배경, 교회사 등)를 **1920×1080 유튜브용 모션 그래픽 MP4**로 만들어 주는 Claude 스킬입니다.
지도 애니메이션 + 무대 연출, 한국어 AI 나레이션(TTS), BGM, 자막까지 한 번에 처리합니다.

> 만든 사람: [jesuswithme](https://github.com/jesuswithme) · 채널 "알려줌"

---

## 설치 방법

### 1) Claude Code (터미널)

```bash
npx degit jesuswithme/skills/motion-graphics-narrated-video ~/.claude/skills/motion-graphics-narrated-video
```

또는 git으로:

```bash
git clone --depth 1 https://github.com/jesuswithme/skills.git /tmp/jw-skills
mkdir -p ~/.claude/skills
cp -r /tmp/jw-skills/motion-graphics-narrated-video ~/.claude/skills/
```

- 특정 프로젝트에서만 쓰려면 `~/.claude/skills/` 대신 프로젝트의 `.claude/skills/`에 넣으세요.
- 설치 후 Claude Code를 다시 시작합니다.

### 2) Claude 앱 (웹 · 데스크톱)

1. 이 `motion-graphics-narrated-video` 폴더를 내려받습니다 (`SKILL.md`가 들어 있는 폴더째로).
2. 폴더를 **zip**으로 압축합니다.
3. Claude 설정 → Capabilities(기능) → Skills에서 zip 파일을 업로드합니다. (메뉴 이름은 앱 버전에 따라 조금 다를 수 있어요.)

---

## 사용 예시

```
바울의 1차 선교 여행 원고야. 이 내용으로 모션 그래픽 영상 만들어줘.
유튜브용으로 다이나믹하게, 나레이션·BGM·자막은 알아서 해줘.
```

옵션도 말로 붙이면 됩니다: `여성 목소리로`, `BGM 1곡만`, `음악 없이`, `3분 이내로`, `9:16 세로로` 등.

---

## 필요한 것 (사전 준비)

| 항목 | 용도 | 설치 예시 |
|---|---|---|
| Python 3 | 스크립트 실행 | – |
| ffmpeg | 영상·음성 합성 | `brew install ffmpeg` / `sudo apt install ffmpeg` |
| edge-tts, shapely, playwright | 나레이션, 지도 데이터, 화면 렌더링 | `pip install edge-tts shapely playwright` |
| Chromium (Playwright) | 프레임 렌더링 | `playwright install chromium` |
| 한글 폰트 (Noto Sans/Serif CJK KR) | 자막·타이틀 | OS별 설치 |
| 인터넷 연결 | 나레이션(Edge TTS), 지도 해안선 데이터(Natural Earth) | – |

---

## ⚠️ 꼭 확인할 점

1. **인증서 패치 줄 지우기**
   `SKILL.md`의 `tts.py` 예시 맨 위에 있는 아래 줄은 **스킬을 만든 특정 클라우드 환경 전용**입니다.
   ```python
   certifi.where = lambda: "/root/.ccr/ca-bundle.crt"
   ```
   일반 PC에서는 이 줄 때문에 나레이션 생성이 실패하니, **이 줄을 빼고** 실행하세요. (Claude에게 "certifi 패치 줄은 빼고 진행해"라고 말해도 됩니다.)

2. **BGM은 TopView MCP 연결이 필요**
   배경음악은 TopView 커넥터로 생성합니다. TopView를 연결하지 않았다면 Claude에게
   `BGM은 내가 준 mp3 파일로 써줘` 또는 `음악 없이`라고 알려 주세요.
   TopView로 만든 음원을 유튜브에 올릴 때는 TopView 이용 조건을 확인하세요.

3. **렌더링 시간**
   프레임 단위로 렌더링하기 때문에 4분 영상 기준 20~40분 정도 걸릴 수 있습니다.

---

## 결과물

- `1920×1080 · 30fps` MP4 (나레이션 + BGM + 한국어 자막)
- 채팅 전송용으로 30MB 이하 압축본 + 고화질 원본(master.mp4)

자세한 제작 절차는 [`SKILL.md`](./SKILL.md)를 참고하세요.
