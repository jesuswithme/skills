---
name: "motion-graphics-narrated-video"
description: "Turn a Korean text (Bible journey, epistle background, history) into a dynamic 1080p YouTube motion-graphics MP4 with map animation, TTS narration, BGM and subtitles."
---

# 나레이션 모션 그래픽 영상 제작 (지도 + 무대 연출)

사용자가 원고(예: 바울의 선교 여행, 서신서 집필 배경, 교회사 사건)를 주고 "이 내용으로 모션 그래픽을 만들어줘 / 유튜브용 / 다이나믹하게 / 나레이션·BGM은 알아서"라고 하면 이 스킬을 따른다. 결과물은 **1920×1080 · 30fps MP4(나레이션 + BGM + 한국어 자막)** 한 개이며, 채팅 전송 한도(30MB) 안에 맞춘 파일을 보낸다.

## 0. 첫 응답
- 한 문장으로 무엇을 만들지 말하고 곧바로 시작한다(질문으로 멈추지 않는다. 사용자가 자리를 비운 새벽 요청이 많다).
- 작업 폴더 예: `/home/claude/<짧은이름>/`, 하위 `tts/`.
- 이전 편과 같은 시리즈면 **같은 비주얼 언어**(아래 7절)를 유지한다. 사용자가 이전 편 캡처를 올리면 그 모양에 맞춘다.
- 사용자 옵션(길이, 목소리, BGM 없음/1곡, 구절 표기 여부, 9:16 세로)이 있으면 따른다.

## 1. 원고 → 장면 설계
1. 원고를 6~10개 장면(scene)으로 나눈다. 전형적 구조:
   - `intro`(질문/카피 → 금색 대제목 글자별 등장)
   - `overview`(전체 경로 한눈에 — 여행 영상일 때)
   - 본론 장면들(도시/목적/편지별 STAGE·PART·PURPOSE·LETTER)
   - `outro`(요약 칩 + 핵심 메시지 타이틀)
2. 각 장면을 **지도 장면**(경로·도시·카메라 이동)과 **무대 장면**(풀스크린 타이포/도식: VS 대결, 도장, 편지 카드, 방패, 식탁 이동 등) 중 하나로 정한다. 추상 개념(교리, 논쟁, 결정)은 무대, 이동·지리는 지도.
3. 나레이션은 문장 단위로 쓴다(한 줄 = 한 TTS 파일 = 자막 한 개 = 애니메이션 비트).
   - 숫자·연도는 **읽는 대로** 쓴다: `제이차`, `서기 오십 년에서 오십삼 년경`, `삼 주`, `일 년 육 개월`.
   - 자막용 표기는 `timeline.py`의 `disp()` 치환표로 되돌린다(`제이차→제2차`, `일 년 육 개월→1년 6개월` …).
   - 원고에 없는 사실은 넣지 않는다. 연출상 넣은 것(성경 구절 표기, 연대, 비유 문구)은 **최종 보고에 목록으로 밝힌다**. 성경 본문을 길게 인용하지 말고 짧게 요지로 쓰며 "요지"라고 표기한다.
   - 시리즈 이전 편과 사실 표기가 다르면 원고를 따르되 보고에서 짚는다.

## 2. BGM (TopView MCP)
- `topview_get_generation_config(type=music)` → 모델 `Topview Music`, `instrumental:true`, `enhancePrompt:true`.
- 기본 **2곡**: 전반(여정/따뜻함/비전) + 후반(긴장·위기 → 승리/통합). 장면 분위기에 맞춘 영어 styles/lyrics 프롬프트. 사용자가 "BGM 1곡"/"음악 없이"라고 하면 따른다.
- 대본 작성과 동시에 제출(병렬), 나중에 `topview_query_task(taskType=ai_music)`로 URL을 받아 `curl -sSL -o bgm1.mp3 <url>`.
- 저장 보드는 기본 'My First Board'. 사용자가 보드를 지정하면 그 보드로.
- 크레딧 사용량(보통 곡당 1)을 최종 보고에 적고, 유튜브 업로드 전 이용 조건 확인을 안내한다.

## 3. 나레이션 TTS (무료 Edge 신경망 음성)
```bash
pip install --break-system-packages -q edge-tts shapely playwright
```
Edge TTS는 프록시 인증서 때문에 certifi를 패치해야 동작한다(`SSL_CERT_FILE`만으로는 안 됨).

`script.json` 형식: `[{"id":"intro","lines":["…","…"]}, …]`

`tts.py`
```python
import certifi, asyncio, json, subprocess
certifi.where = lambda: "/root/.ccr/ca-bundle.crt"
import edge_tts
S=json.load(open("script.json"))
VOICE="ko-KR-InJoonNeural"   # 여성: ko-KR-SunHiNeural
async def one(i,j,txt):
    fn=f"tts/{i:02d}_{j:02d}.mp3"
    for k in range(3):
        try:
            await edge_tts.Communicate(txt,VOICE,rate="+6%").save(fn); return fn
        except Exception as e: print("retry",e)
async def main():
    out=[]
    for i,s in enumerate(S):
        for j,l in enumerate(s["lines"]):
            fn=await one(i,j,l)
            d=float(subprocess.check_output(["ffprobe","-v","0","-show_entries","format=duration","-of","csv=p=0",fn]))
            out.append({"scene":s["id"],"si":i,"li":j,"text":l,"file":fn,"dur":d})
    json.dump(out,open("lines.json","w"),ensure_ascii=False,indent=1)
    print(sum(o["dur"] for o in out))
asyncio.run(main())
```
(Edge TTS가 막히면 gTTS가 대안이지만 음질이 떨어진다.)

## 4. 타임라인
`timeline.py` — 장면 시작 여유(LEAD), 줄 간격(GAP), 장면 끝 여유(TAIL)를 두고 각 줄의 시작 `t`/끝 `e`를 계산해 `tl.json`으로 저장. 모든 애니메이션은 이 시간에 묶는다.
```python
import json
def disp(s):
    for a,b in [("제이차","제2차"),("일 년 육 개월","1년 6개월")]: s=s.replace(a,b)  # 영상마다 수정
    return s
L=json.load(open("lines.json"))
LEAD={"intro":2.2,"overview":1.0}; GAP=0.45; TAIL={"outro":3.5}
scenes=[];t=0.0;cur=None
for l in L:
    if cur is None or cur["id"]!=l["scene"]:
        if cur: cur["end"]=t+TAIL.get(cur["id"],0.7); t=cur["end"]
        cur={"id":l["scene"],"start":t,"lines":[]}; scenes.append(cur)
        t+=LEAD.get(l["scene"],1.3)
    cur["lines"].append({"t":round(t,3),"e":round(t+l["dur"],3),"text":disp(l["text"]),"file":l["file"]})
    t+=l["dur"]+GAP
cur["end"]=t+TAIL.get(cur["id"],0.7)
json.dump(scenes,open("tl.json","w"),ensure_ascii=False,indent=1)
for s in scenes: print(s["id"],round(s["start"],1),round(s["end"],1))
```

## 5. 지도 데이터 (실제 해안선)
```bash
curl -sSL -o ../land.geojson https://raw.githubusercontent.com/nvkelso/natural-earth-vector/master/geojson/ne_10m_land.geojson
curl -sSL -o ../rivers.geojson https://raw.githubusercontent.com/nvkelso/natural-earth-vector/master/geojson/ne_10m_lakes.geojson
```
`geo.py`: 등장방형 투영 `P(lon,lat)=((lon-LON0)*0.809*240, (LAT1-lat)*240)`. bbox를 **카메라가 볼 수 있는 범위보다 넉넉히**(가장자리가 비면 바다색 띠가 보임) 잡고 shapely로 clip+simplify 해서 SVG path 문자열을 만든다.
- `land`(tolerance ~0.006) + **`landlo`(tolerance ~0.03, minarea 0.02)** 두 벌. 줌이 0.75 미만일 때 `landlo`로 교체(LOD) — 넓은 지도 장면 렌더링 속도가 2~3배 빨라진다.
- `cities`(도시 lon/lat → P), `ways`(바닷길은 섬·반도를 피하는 경유점 배열), `labels`(지역명 위치).
- 주요 좌표 예: 수리아 안디옥(36.16,36.20) 실루기아(35.93,36.12) 다소(34.90,36.92) 더베(33.37,37.35) 루스드라(32.45,37.58) 이고니온(32.48,37.87) 비시디아 안디옥(31.19,38.31) 버가(30.85,36.96) 앗달리아(30.70,36.89) 살라미(33.90,35.18) 바보(32.42,34.76) 드로아(26.16,39.75) 네압볼리(24.40,40.94) 빌립보(24.29,41.01) 데살로니가(22.94,40.64) 베뢰아(22.20,40.52) 아덴(23.73,37.98) 고린도(22.88,37.91) 겐그레아(23.00,37.88) 에베소(27.34,37.94) 가이사랴(34.89,32.50) 예루살렘(35.23,31.78) 로마(12.49,41.89) 서바나/타라코(1.25,41.12).

## 6. HTML 템플릿 (결정론적 `setTime(t)`)
`tpl.html`에 `__GEO__`, `__TL__` 자리표시자를 두고 `build.py`로 치환해 `index.html`을 만든다.
```python
s=open("tpl.html").read().replace("__GEO__",open("geo.json").read()).replace("__TL__",open("tl.json").read())
open("index.html","w").write(s)
```
핵심 원칙:
- **CSS transition/animation 금지.** 모든 상태는 `window.setTime(t)`가 t로부터 계산(프레임 단위 렌더와 구간 재렌더가 가능해야 함).
- 헬퍼: `P(t,a,b)`(0~1 진행), `eo/eio/eback` 이징, `win(t,a,b,fi,fo)` 페이드 창, `LT(scene,i)/LE/SC/SE` 타임라인 조회.
- 레이어 순서: 바다 그라디언트 → `<g id=cam>`(격자, 육지 그림자·육지·해칭, 호수, 지역명, 경로, 도시, 이동 아이콘) → 파티클 canvas → `#stage`(풀스크린 무대) → HUD/카드 → `#over`(스팅어·말풍선) → 자막 → 플래시/블랙/비네트.
- **카메라**: 키프레임 `cam(t,dur,x,y,zoom,anchorX,anchorY)`, 줌은 로그 보간. 오른쪽에 정보 카드가 있으면 anchorX≈700. 화면 흔들림(shake)은 위기 장면에만 짧게.
- **경로(leg)**: mask path의 `stroke-dashoffset`으로 그려지는 효과. 색 규칙 — 육로 금색, 해로 하늘색 점선+⛵, 이탈/박해 빨강 점선, 귀환·통합 청록, 편지 보라+📜, 비전 연한 금색 점선. 진행점에 이동 아이콘(`getPointAtLength`).
- 도시·라벨은 `scale(1/zoom)`으로 화면 크기 일정, 라벨 배경 rect는 `getBBox`로 맞춤. 등장 시 `eback` 팝 + 맥동 링.
- 텍스트 요소 대부분에 `white-space:nowrap`/`word-break:keep-all`. 두 줄이 필요하면 줄을 **명시적으로 나눠** 넣는다(예: "고린도에서 보낸" / "석 달").
- `.gold`(background-clip:text) 안에 이모지를 넣지 않는다(금색 네모로 깨짐). 이모지는 span 밖에.

## 7. 비주얼 언어 (시리즈 공통)
- 팔레트: 밤바다 네이비 #050b16, 모래색 육지, 금색 #f0c46e/#f7d28a, 청록 #35d0c4, 빨강 #ff5a5a. 폰트 Noto Sans/Serif CJK KR.
- 상단 HUD(좌: 영문 키커+한글 제목, 우: STAGE/PART/PURPOSE/LETTER 진행 표시), 오른쪽 정보 카드(배지·제목·영문·항목이 나레이션에 맞춰 하나씩 등장), 하단 자막 박스.
- **스팅어**: 장면 시작 2.4초, 가로 띠 + "STAGE 03 / 빌립보" 슬라이드. 무대 장면의 자체 타이틀과 겹치면 스팅어를 빼거나 타이틀을 늦춘다.
- 팀 HUD(인물 토큰 합류 NEW·잔류 태그·이름 변경 등)로 인물 변화를 보여 준다.
- 자주 쓴 무대 연출: 인용문 타이핑, VS 분할 패널+번개 균열, 빨간 도장(추방/폭동/가짜 사도?), 금색 도장(오직 은혜로 구원/타협 없음), 편지 카드+밀랍 봉인, 두루마리 공문, 감옥 창살이 날아가는 지진, 환상(별밤+빛나는 인물), 카운터(3주, 1년 6개월, 1~3월), 비교 카드(갈라디아서 vs 로마서), 받침대 높이가 같아지는 "모두 죄인", 두 원이 하나로 합쳐짐, 방사형 빛(conic), 에필로그 빛줄기/요약 칩.

## 8. 점검 (렌더 전 필수)
`snap.py t1 t2 …`로 특정 시각을 캡처하고, PIL로 960×540 썸네일 4~6장을 한 이미지로 붙여 `Read`로 확인한다. 콘솔 에러도 출력.
```python
import sys,asyncio,os
from playwright.async_api import async_playwright
async def main():
    async with async_playwright() as p:
        b=await p.chromium.launch(); pg=await b.new_page(viewport={"width":1920,"height":1080})
        errs=[];pg.on("pageerror",lambda e:errs.append(str(e)))
        await pg.goto("file://"+os.path.abspath("index.html"));await pg.wait_for_timeout(500)
        for t in sys.argv[1:]:
            await pg.evaluate(f"setTime({t})");await pg.screenshot(path=f"snap_{t}.jpg",type="jpeg",quality=80)
        print(errs[:5]); print(await pg.evaluate("TOTAL"))
        await b.close()
asyncio.run(main())
```
체크리스트: 글자 줄바꿈 깨짐(한 글자만 다음 줄), 라벨·말풍선이 자막/카드/HUD와 겹침, 오버레이 두 개 동시 표시, 카메라가 목적지를 벗어남, 경로가 섬·반도를 가로지름, 지도 가장자리 빈 띠, 무대 전환 사이 빈 화면, 원고와 다른 사실.

## 9. 렌더링
`render.py i0 i1 out.mp4` — CDP 스크린샷(JPEG q92)을 ffmpeg 파이프로 x264 crf18.
```python
import sys,asyncio,subprocess,time,base64,os
from playwright.async_api import async_playwright
FPS=30
async def main(i0,i1,out):
    async with async_playwright() as p:
        b=await p.chromium.launch(); pg=await b.new_page(viewport={"width":1920,"height":1080})
        await pg.goto("file://"+os.path.abspath("index.html"));await pg.wait_for_timeout(800)
        ff=subprocess.Popen(["ffmpeg","-y","-loglevel","error","-f","image2pipe","-framerate",str(FPS),"-c:v","mjpeg","-i","-","-c:v","libx264","-preset","veryfast","-crf","18","-pix_fmt","yuv420p",out],stdin=subprocess.PIPE)
        cdp=await pg.context.new_cdp_session(pg); t0=time.time()
        for f in range(i0,i1):
            await pg.evaluate(f"setTime({f/FPS})")
            r=await cdp.send("Page.captureScreenshot",{"format":"jpeg","quality":92})
            ff.stdin.write(base64.b64decode(r["data"]))
            if f%300==0: print(out,f,round(time.time()-t0,1),flush=True)
        ff.stdin.close();ff.wait();await b.close()
asyncio.run(main(int(sys.argv[1]),int(sys.argv[2]),sys.argv[3]))
```
- 총 프레임 N=ceil(마지막 장면 end×30)을 반으로 나눠 **백그라운드 2개 병렬**(`(nohup python3 render.py 0 H part1.mp4 > r1.log 2>&1 &)`). 속도는 1~3fps, 4분 영상에 20~40분.
- 진행 확인은 `sleep 590` 후 로그 tail. **`pgrep -f render.py`/`pkill -f`를 쓰지 않는다** — 명령줄 자체가 매칭돼 무한 대기나 셸 종료가 난다. `ps aux | grep -c "[r]ender.py"`를 쓴다.
- 렌더 중 사용자에게 `SendUserMessage`로 구성 미리보기와 예상 시간을 한 번 알린다.

## 10. 오디오 믹스
`mix.py`: 각 줄 음성을 시작 시각에 배치, 발화 마스크를 이동평균(cumsum 방식 — `np.convolve`는 너무 느림)으로 부드럽게 해 BGM 더킹(발화 중 약 0.27, 공백 0.85), BGM1→BGM2를 장면 전환점 X에서 XF초 크로스페이드(두 곡 길이로 X 범위 확인), 페이드인/아웃, 음성 RMS 0.16 정규화, BGM ×0.7, `loudnorm=I=-15:TP=-1.5` → `mix.m4a`.
```python
import json,subprocess,numpy as np
SR=48000
def load(fn):
    raw=subprocess.check_output(["ffmpeg","-v","0","-i",fn,"-f","f32le","-ac","2","-ar",str(SR),"-"])
    return np.frombuffer(raw,np.float32).reshape(-1,2).copy()
TL=json.load(open("tl.json")); END=TL[-1]["end"]; N=int(END*SR)+SR
voice=np.zeros((N,2),np.float32); speech=np.zeros(N,np.float32)
for s in TL:
    for l in s["lines"]:
        a=load(l["file"]); i=int(l["t"]*SR); voice[i:i+len(a)]+=a; speech[i:i+len(a)]=1
def ma(x,k):
    c=np.cumsum(np.concatenate([[0],x])); h=k//2
    i0=np.clip(np.arange(len(x))-h,0,len(x)); i1=np.clip(np.arange(len(x))+h,0,len(x))
    return (c[i1]-c[i0])/k
k=int(0.35*SR); m=ma(ma(speech,k),k); gain=0.85-0.58*np.clip(m*1.6,0,1)
b1=load("bgm1.mp3"); b2=load("bgm2.mp3"); bgm=np.zeros((N,2),np.float32)
X=100.0; XF=6.0   # 영상마다: 분위기가 바뀌는 장면 시작 직전
n1=min(len(b1),int((X+XF)*SR)); seg=b1[:n1].copy(); t=np.arange(n1)/SR; seg*=np.clip((X+XF-t)/XF,0,1)[:,None]; bgm[:n1]+=seg
i2=int(X*SR); seg2=b2[:N-i2].copy(); t2=np.arange(len(seg2))/SR; seg2*=np.clip(t2/4.0,0,1)[:,None]; bgm[i2:i2+len(seg2)]+=seg2
tt=np.arange(N)/SR; bgm*=(gain*np.clip(tt/1.5,0,1)*np.clip((END-0.2-tt)/3.5,0,1))[:,None]
voice*=0.16/np.sqrt(np.mean(voice[speech>0]**2))
out=voice+bgm*0.7; pk=np.abs(out).max()
if pk>0.97: out*=0.97/pk
out=out[:int(END*SR)]
subprocess.run(["ffmpeg","-y","-v","error","-f","f32le","-ar",str(SR),"-ac","2","-i","-","-af","loudnorm=I=-15:TP=-1.5:LRA=11","-ar","48000","-c:a","aac","-b:a","256k","mix.m4a"],input=out.tobytes(),check=True)
```

## 11. 합치기 · 압축 · 전달
```bash
printf "file part1.mp4\nfile part2.mp4\n" > list.txt
ffmpeg -y -v error -f concat -safe 0 -i list.txt -i mix.m4a -map 0:v -map 1:a -c copy -shortest -movflags +faststart master.mp4
# 30MB 한도용 2-pass: 비디오 kbps ≈ (29*8*1024/길이초) - 오디오kbps  (보통 700k~1200k)
ffmpeg -y -v error -i master.mp4 -c:v libx264 -preset slow -b:v 900k -pass 1 -an -f null /dev/null
ffmpeg -y -v error -i master.mp4 -c:v libx264 -preset slow -b:v 900k -pass 2 -c:a aac -b:a 128k -movflags +faststart "<한글제목>.mp4"
```
- `SendUserFile`은 30 MiB 초과 시 거부된다 → 반드시 2-pass로 맞춘다. 원본(master) 크기도 보고한다.
- 완성본에서 8~12프레임을 뽑아 격자로 최종 확인 후 전송(`display:"render"`).

## 12. 부분 수정(재렌더 구간 교체)
문제 구간만 `render.py a b fix.mp4`로 다시 뽑고, **같은 파일을 두 번 입력**해 trim으로 이어 붙인 뒤(한 입력을 두 번 trim하면 메모리 폭주로 OOM) 다시 2-pass:
```bash
ffmpeg -y -v error -f concat -safe 0 -i list.txt -c copy full_old.mp4
ffmpeg -y -v error -i full_old.mp4 -i full_old.mp4 -i fix.mp4 -filter_complex "[0:v]trim=end_frame=A,setpts=PTS-STARTPTS[a];[1:v]trim=start_frame=B,setpts=PTS-STARTPTS[b];[a][2:v][b]concat=n=3:v=1[v]" -map "[v]" -c:v libx264 -preset veryfast -crf 16 -pix_fmt yuv420p spliced.mp4
```
이음매 전후 프레임을 캡처해 확인한다.

## 13. 최종 보고 (한국어, 간결)
- 무엇이 나왔는지(길이·해상도), 장면 흐름 요약(굵게 핵심 연출)
- BGM: TopView 곡 수·크레딧, 전환 지점, 이용 조건 확인 안내
- **원고에 없던 연출 추가 사항**(구절 표기, 연대, 비유, 해석) 목록 + 이전 편과 사실 표기가 다르면 짚기
- 알려진 흠(잘림·겹침)과 구간 재렌더 제안, 원본 파일 크기