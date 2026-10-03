# 🛠️ VRChat / Unity 로고 배치 가이드
### VRChat / Unity Logo Placement Guide

> 본 문서는 **VRChat 한국수어교실** 공식 브랜드 자산을 VRChat 월드 및 Unity 씬에 배치할 때
> **로고가 어두워지거나, 찌그러지거나, 뭉개지는 현상**을 방지하기 위한 기술 가이드입니다.
>
> *This document explains how to place the official brand assets in VRChat worlds and Unity scenes without darkening, stretching, or blurring the logo.*

> [!IMPORTANT]
> ### 🔐 배치 전 필수 확인 (Before You Deploy)
>
> 본 가이드는 **이미 사용 승인을 받은 담당자**를 위한 기술 문서입니다.
> **총괄 대표(이심율)의 사전 서면 승인 없이는 어떠한 월드에도 자산을 배치할 수 없습니다.**
>
> 👉 [공식 사용 승인 신청하기](https://github.com/LeeSimYul/Kidentity/issues/new?template=brand_authorization_request.md)
>
> *This is a technical document for handlers who have already obtained authorization.*
> ***No asset may be placed in any world without the Lead Director's prior written approval.***

---

## 📑 목차 (Table of Contents)

1. [권장 자산 선택](#sec1)
2. [텍스처 임포트 설정](#sec2)
3. [Unlit 셰이더 설정 — 시인성 확보](#sec3)
4. [Translucent / 반투명 설정](#sec4)
5. [텍스처 찌그러짐 방지 — 비율 계산](#sec5)
6. [거리별 선명도 — Mipmap & Aniso](#sec6)
7. [라이트맵 · 최적화 체크리스트](#sec7)
8. [문제 해결 (Troubleshooting)](#sec8)
9. [English Guide](#en)

---

<a name="sec1"></a>

## 1. 권장 자산 선택 (Choosing the Right Asset)

| 배치 상황 | 권장 파일 | 이유 |
| :--- | :--- | :--- |
| **어두운 배경 벽면** | `타이틀 배너(투명화).png` | 투명 배경으로 배경색과 자연스럽게 융합 |
| **밝은 배경 · 흰 벽면** | `공식 타이틀 배너 together.png` | 배경 포함 버전이 대비를 확보 |
| **원형 아이콘 · 포털 썸네일** | `logo-icon/로고 ver.2 (투명배경화).png` | 정사각 비율에 최적화 |
| **대형 출력 · 초근접 배치** | `assets/svg/*.svg` (PNG로 재출력) | 벡터에서 필요한 해상도로 래스터화 |

> [!CAUTION]
> **AI · SVG 원본 파일은 편집용 마스터 소스입니다.**
> 승인된 담당자 외 외부 반출이 금지되며, 외주 제작자에게 전달하려면 **반드시 사전 승인**을 받아야 합니다.
> 또한 벡터에서 재출력하더라도 **요소 분리(지화 그래픽·손모양 아이콘만 추출), 색상 변경, 비율 변형은 금지**됩니다.

---

<a name="sec2"></a>

## 2. 텍스처 임포트 설정 (Texture Import Settings)

Unity `Project` 창에서 로고 PNG를 선택한 뒤, `Inspector`에서 아래와 같이 설정합니다.

| 항목 | 권장 값 | 설명 |
| :--- | :--- | :--- |
| **Texture Type** | `Default` | Sprite(2D)가 아닌 월드 배치용 기본 타입 |
| **Alpha Source** | `Input Texture Alpha` | PNG의 투명도를 그대로 사용 |
| **Alpha Is Transparency** | ✅ **체크** | **외곽선 검은 테두리 방지의 핵심 옵션** |
| **sRGB (Color Texture)** | ✅ 체크 | 색상 텍스처이므로 반드시 체크 |
| **Wrap Mode** | `Clamp` | 가장자리 픽셀 반복으로 생기는 얇은 선 방지 |
| **Filter Mode** | `Bilinear` (또는 `Trilinear`) | 확대 시 계단 현상 완화 |
| **Generate Mip Maps** | ✅ 체크 | 원거리 노이즈(지글거림) 방지 |
| **Aniso Level** | `4` ~ `8` | 비스듬히 볼 때 선명도 유지 |
| **Max Size** | `2048` (대형 배너는 `4096`) | 과도한 해상도는 용량만 증가 |
| **Compression** | `High Quality` | 로고 외곽선 깨짐 최소화 |
| **Format** | `Auto` / PC: `DXT5(BC3)` | 알파 채널 보존 필수 |

> [!TIP]
> **`Alpha Is Transparency`를 반드시 체크하세요.**
> 이 옵션이 꺼져 있으면 투명 영역의 RGB 값이 검은색으로 채워져,
> 로고 외곽선 주변에 **검은 테두리(halo)가 번지는 현상**이 발생합니다.

---

<a name="sec3"></a>

## 3. Unlit 셰이더 설정 — 시인성 확보 (Unlit Shader)

VRChat 월드는 라이팅 환경이 월드마다 다릅니다. **Standard 셰이더로 배치하면**
어두운 월드에서 로고가 **거의 보이지 않을 정도로 어두워집니다.**
브랜드 로고는 **조명의 영향을 받지 않는 Unlit 셰이더**로 배치해야 원본 색상이 그대로 유지됩니다.

### 3-1. 머티리얼 생성

```
Project 창 우클릭 ➔ Create ➔ Material ➔ 이름: M_KSL_Logo_Unlit
```

### 3-2. 셰이더 선택

| 용도 | 셰이더 경로 | 특징 |
| :--- | :--- | :--- |
| **불투명 배너 (배경 포함)** | `Unlit/Texture` | 가장 가볍고 안정적 |
| **투명 PNG 로고 (권장)** | `Unlit/Transparent` | 알파 채널 지원, 반투명 표현 가능 |
| **외곽선을 또렷하게** | `Unlit/Transparent Cutout` | 알파 경계를 잘라내어 깔끔한 외곽선 |
| **색상 조정이 필요할 때** | `Unlit/Color` + Texture | Tint 컬러 곱연산 지원 |

> [!NOTE]
> **`Unlit/Transparent Cutout` 사용 시** `Alpha Cutoff` 값을 `0.3` ~ `0.5`로 조정하세요.
> 값이 너무 낮으면 반투명 잔상이 남고, 너무 높으면 얇은 획이 끊어집니다.

### 3-3. 머티리얼 설정값

```
Shader          : Unlit/Transparent
Base (RGB)      : [로고 PNG 파일 드래그]
Tint Color      : 흰색 (255, 255, 255, 255)  ← 색상 변경 금지 조항 준수
Render Queue    : Transparent (3000)
```

> [!WARNING]
> **Tint Color를 흰색 이외의 값으로 변경하지 마십시오.**
> 머티리얼 Tint를 이용한 색상 변경 역시 브랜드 가이드라인의 **「색상 임의 변경 금지」 조항 위반**에 해당합니다.

### 3-4. 렌더러 설정 (Mesh Renderer)

로고를 배치한 Quad의 `Mesh Renderer` 컴포넌트에서:

| 항목 | 권장 값 | 이유 |
| :--- | :--- | :--- |
| **Cast Shadows** | `Off` | Unlit 평면이 그림자를 드리울 필요 없음 |
| **Receive Shadows** | ☐ 해제 | 다른 오브젝트 그림자로 로고가 어두워지는 것 방지 |
| **Contribute Global Illumination** | ☐ 해제 | 라이트맵 베이크 시 로고 얼룩짐 방지 |
| **Light Probes** | `Off` | Unlit은 프로브 영향을 받지 않음 |

---

<a name="sec4"></a>

## 4. Translucent / 반투명 설정 (Transparency)

벽면 사이니지나 홀로그램 연출처럼 **은은한 반투명 로고**가 필요한 경우입니다.

### 4-1. 알파값으로 투명도 조절

`Unlit/Transparent` 셰이더에서 **Tint Color의 A(Alpha) 값만** 조절합니다.

| 용도 | Alpha 권장값 |
| :--- | :---: |
| 메인 타이틀 배너 (완전 불투명) | `255` (1.0) |
| 벽면 보조 사이니지 | `180` ~ `220` |
| 바닥 · 천장 워터마크 | `100` ~ `140` |
| 홀로그램 연출 | `120` ~ `160` |

> [!CAUTION]
> **로고의 시인성은 브랜드 공신력과 직결됩니다.**
> Alpha `100` 미만의 과도한 반투명 처리는 로고 판독을 어렵게 하므로 권장하지 않습니다.

### 4-2. 반투명 오브젝트 정렬 문제 (Z-Sorting)

반투명 오브젝트가 겹치면 앞뒤 순서가 뒤집혀 보일 수 있습니다.

| 해결 방법 | 설정 |
| :--- | :--- |
| 렌더 큐 수동 조정 | 머티리얼 `Render Queue`를 `3001`, `3002`… 로 단계 지정 |
| 벽면에서 살짝 띄우기 | 벽에서 **0.01 ~ 0.02 유닛** 앞으로 배치 (Z-fighting 방지) |
| 불필요한 겹침 제거 | 반투명 평면을 서로 겹치지 않게 배치 |

---

<a name="sec5"></a>

## 5. 텍스처 찌그러짐 방지 — 비율 계산 (Aspect Ratio)

> [!WARNING]
> ### ⛔️ 비율 왜곡은 라이선스 위반입니다
> 로고의 **가로/세로 비율을 임의로 변형(찌그러뜨림)하는 행위**는
> CC BY-NC-ND 4.0의 **변경 금지(ND)** 조항 및 브랜드 가이드라인 위반에 해당합니다.

### 5-1. 왜 찌그러지는가

Unity의 **Quad 기본 크기는 1 × 1 정사각형**입니다.
가로로 긴 배너(예: 1920 × 640)를 그대로 Quad에 올리면 **세로로 눌린 형태**가 됩니다.

### 5-2. 올바른 스케일 계산 공식

```
가로 기준 배치:
  Scale X = 원하는 가로 길이 (W)
  Scale Y = W × (텍스처 세로 픽셀 ÷ 텍스처 가로 픽셀)
  Scale Z = 1  (Quad는 두께가 없으므로 항상 1)
```

### 5-3. 계산 예시

| 텍스처 원본 해상도 | 비율 | 원하는 가로 | **정확한 Scale 값** |
| :--- | :---: | :---: | :--- |
| 1920 × 640 | 3 : 1 | 6 m | `X=6, Y=2, Z=1` |
| 1600 × 900 | 16 : 9 | 4 m | `X=4, Y=2.25, Z=1` |
| 1024 × 1024 | 1 : 1 | 2 m | `X=2, Y=2, Z=1` |
| 2048 × 512 | 4 : 1 | 8 m | `X=8, Y=2, Z=1` |

> [!TIP]
> **텍스처 해상도 확인 방법**
> `Project` 창에서 PNG 선택 ➔ `Inspector` 하단 미리보기 영역에 `1920x640` 형태로 표시됩니다.
>
> **검증 방법**
> `Scale X ÷ Scale Y` 값과 `텍스처 가로 ÷ 텍스처 세로` 값이 **같으면 비율이 정확**합니다.

### 5-4. 자주 하는 실수

| ❌ 잘못된 방식 | ✅ 올바른 방식 |
| :--- | :--- |
| Scene 뷰에서 스케일 핸들을 눈대중으로 드래그 | Inspector에 계산된 수치를 직접 입력 |
| `X=5, Y=5`처럼 정사각형으로 맞춤 | 텍스처 비율에 맞춰 Y값 계산 |
| 부모 오브젝트에 비균등 스케일 적용 | 부모 스케일은 `1,1,1` 유지 후 자식에서 조정 |
| Canvas UI로 배치 후 늘림 | 월드 배치는 Quad + Unlit 권장 |

> [!CAUTION]
> **부모 오브젝트의 비균등 스케일에 주의하세요.**
> 부모가 `X=2, Y=1`처럼 비균등 스케일을 가지면 자식 Quad의 비율이 정확해도 **최종 렌더링에서 찌그러집니다.**
> 반드시 부모 Transform의 Scale을 `1, 1, 1`로 유지하십시오.

---

<a name="sec6"></a>

## 6. 거리별 선명도 — Mipmap & Aniso

VRChat 월드에서는 플레이어가 로고를 **멀리서, 비스듬히** 보는 경우가 많습니다.

| 증상 | 원인 | 해결 |
| :--- | :--- | :--- |
| 멀리서 보면 지글거림 | Mipmap 미생성 | `Generate Mip Maps` ✅ 체크 |
| 멀리서 보면 뿌옇게 뭉개짐 | Mipmap 과도 적용 | `Mip Map Filtering: Kaiser`, Bias 값 `-0.5` |
| 비스듬히 보면 흐려짐 | 이방성 필터링 부족 | `Aniso Level`을 `4`~`8`로 상향 |
| 가장자리에 얇은 선 | Wrap Mode 반복 | `Wrap Mode: Clamp` 설정 |
| 확대 시 계단 현상 | 해상도 부족 | 벡터(SVG/AI)에서 더 높은 해상도로 재출력 |

---

<a name="sec7"></a>

## 7. 라이트맵 · 최적화 체크리스트 (Optimization)

배치 완료 후 아래 항목을 점검하세요.

- [ ] 머티리얼 셰이더가 `Unlit/*` 계열로 설정되어 있는가
- [ ] `Alpha Is Transparency`가 체크되어 있는가 (검은 테두리 방지)
- [ ] Quad Scale이 텍스처 비율과 정확히 일치하는가
- [ ] 부모 오브젝트의 Scale이 `1, 1, 1`인가
- [ ] `Contribute Global Illumination`이 해제되어 있는가 (라이트맵 얼룩 방지)
- [ ] `Cast Shadows`가 `Off`인가
- [ ] 벽면에서 `0.01` 유닛 이상 띄워 Z-fighting을 방지했는가
- [ ] Tint Color가 흰색(색상 미변형) 상태인가
- [ ] 텍스처 `Max Size`가 과도하지 않은가 (월드 용량 최적화)
- [ ] **월드 설명란에 출처 표기 문구를 기재했는가**
- [ ] **총괄 대표의 사전 서면 승인을 받았는가**

### 출처 표기 문구 (월드 설명란 복사용)

```text
Logo & Brand Identity by VRChat 한국수어교실 (Lead Director: 이심율)
Official Repository: https://github.com/LeeSimYul/Kidentity
Licensed under CC BY-NC-ND 4.0 / Used with prior written approval.
```

---

<a name="sec8"></a>

## 8. 문제 해결 (Troubleshooting)

| 증상 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| 로고가 어둡게 보임 | Standard 셰이더의 라이팅 영향 | 셰이더를 `Unlit/Transparent`로 변경 |
| 외곽선에 검은 테두리 | `Alpha Is Transparency` 미체크 | 텍스처 설정에서 체크 후 `Apply` |
| 투명 영역이 흰색/검은색 | 알파 채널 없는 PNG 사용 | `(투명화)` 버전 파일로 교체 |
| 로고가 옆으로 늘어남 | Quad Scale 비율 불일치 | [5장 공식](#sec5)으로 재계산 |
| 벽면과 겹쳐 깜빡임 | Z-fighting | 벽에서 `0.01`~`0.02` 유닛 띄우기 |
| 뒷면에서 안 보임 | Quad 단면 렌더링 | Quad를 180° 회전 복제하여 뒷면 추가 |
| 라이트맵 베이크 후 얼룩 | GI 기여 설정 | `Contribute Global Illumination` 해제 |
| 반투명 로고 순서 뒤집힘 | 렌더 큐 정렬 | `Render Queue`를 `3001`+ 로 수동 조정 |
| 업로드 후 화질 저하 | 압축 포맷 문제 | `Compression: High Quality`, 알파 보존 포맷 사용 |

---
---

<a name="en"></a>

# 🌏 English Guide

> This is the English translation of the Korean guide above.
> **The Korean text is the official and authoritative version.**

> [!IMPORTANT]
> This is a technical document for handlers who have **already obtained authorization**.
> **No asset may be placed in any world without the prior written approval of the Lead Director (LeeSimYul).**
> 👉 [Submit an authorization request](https://github.com/LeeSimYul/Kidentity/issues/new?template=brand_authorization_request.md)

## 1. Choosing the Right Asset

| Situation | Recommended file | Why |
| :--- | :--- | :--- |
| Dark background wall | Transparent title banner PNG | Blends naturally with the background |
| Light / white wall | Official title banner PNG | The baked background secures contrast |
| Round icon, portal thumbnail | Transparent logo icon PNG | Optimized for a square ratio |
| Large print / very close placement | SVG vector (re-export to PNG) | Rasterize from vector at the needed resolution |

> [!CAUTION]
> **AI and SVG files are editable master sources.** Distribution outside the approved handler is prohibited, and handing them to an outsourced creator requires **prior approval**.
> Even when re-exporting from vector, **element separation, recoloring, and ratio changes remain prohibited.**

## 2. Texture Import Settings

| Setting | Recommended | Notes |
| :--- | :--- | :--- |
| Texture Type | `Default` | Not Sprite(2D) — this is a world placement |
| Alpha Source | `Input Texture Alpha` | Use the PNG's own alpha |
| **Alpha Is Transparency** | ✅ **On** | **Key option that prevents black halos** |
| sRGB (Color Texture) | ✅ On | Required for color textures |
| Wrap Mode | `Clamp` | Prevents thin edge lines from tiling |
| Filter Mode | `Bilinear` / `Trilinear` | Softens aliasing when scaled up |
| Generate Mip Maps | ✅ On | Prevents shimmering at a distance |
| Aniso Level | `4` – `8` | Keeps sharpness at grazing angles |
| Max Size | `2048` (`4096` for large banners) | Excess resolution only costs file size |
| Compression | `High Quality` | Minimizes edge artifacts |
| Format | `Auto` / PC `DXT5(BC3)` | Must preserve the alpha channel |

> [!TIP]
> **Always enable `Alpha Is Transparency`.** With it off, Unity fills transparent pixels' RGB with black, producing a **dark halo bleeding around the logo's edges**.

## 3. Unlit Shader — Securing Visibility

Lighting differs from world to world in VRChat. With the **Standard shader the logo can become almost invisible** in a dark world. Use an **Unlit shader**, which ignores scene lighting, so the original colors are preserved exactly.

**Create the material:** `Project ➔ Create ➔ Material ➔ M_KSL_Logo_Unlit`

| Use case | Shader | Characteristics |
| :--- | :--- | :--- |
| Opaque banner (with background) | `Unlit/Texture` | Lightest and most stable |
| Transparent PNG logo (recommended) | `Unlit/Transparent` | Alpha support, translucency possible |
| Crisp edges | `Unlit/Transparent Cutout` | Clips the alpha edge for a clean outline |
| Needs tint control | `Unlit/Color` + texture | Multiplies a tint color |

> [!NOTE]
> With `Unlit/Transparent Cutout`, set `Alpha Cutoff` between `0.3` and `0.5`. Too low leaves translucent fringes; too high breaks thin strokes.

```
Shader       : Unlit/Transparent
Base (RGB)   : [drag the logo PNG here]
Tint Color   : White (255, 255, 255, 255)   <- keeps the no-recoloring rule
Render Queue : Transparent (3000)
```

> [!WARNING]
> **Never set Tint Color to anything other than white.** Recoloring through the material tint also violates the **"no arbitrary color change"** clause of the brand guidelines.

**Mesh Renderer:** `Cast Shadows: Off`, `Receive Shadows: off`, `Contribute Global Illumination: off`, `Light Probes: Off`.

## 4. Translucent Setup

Adjust only the **A (alpha) channel of the Tint Color** on an `Unlit/Transparent` material.

| Use case | Recommended alpha |
| :--- | :---: |
| Main title banner (fully opaque) | `255` (1.0) |
| Secondary wall signage | `180` – `220` |
| Floor / ceiling watermark | `100` – `140` |
| Hologram effect | `120` – `160` |

> [!CAUTION]
> **Legibility is inseparable from brand credibility.** Alpha values below `100` make the logo hard to read and are not recommended.

For overlapping translucent planes, set `Render Queue` manually (`3001`, `3002`, …) and offset the plane **0.01–0.02 units** off the wall to avoid Z-fighting.

## 5. Preventing Texture Distortion — Aspect Ratio

> [!WARNING]
> **Distorting the logo's aspect ratio violates the NoDerivatives (ND) clause of CC BY-NC-ND 4.0 and the brand guidelines.**

Unity's Quad is **1 × 1 by default**, so a wide banner (e.g. 1920 × 640) dropped straight onto it is **squashed vertically**.

```
Width-first placement:
  Scale X = desired width (W)
  Scale Y = W x (texture height px / texture width px)
  Scale Z = 1   (a Quad has no thickness)
```

| Texture resolution | Ratio | Desired width | **Correct scale** |
| :--- | :---: | :---: | :--- |
| 1920 × 640 | 3 : 1 | 6 m | `X=6, Y=2, Z=1` |
| 1600 × 900 | 16 : 9 | 4 m | `X=4, Y=2.25, Z=1` |
| 1024 × 1024 | 1 : 1 | 2 m | `X=2, Y=2, Z=1` |
| 2048 × 512 | 4 : 1 | 8 m | `X=8, Y=2, Z=1` |

> [!TIP]
> Check the resolution in the `Inspector` preview of the PNG (shown as `1920x640`).
> **Verify:** `Scale X / Scale Y` must equal `texture width / texture height`.

| ❌ Wrong | ✅ Right |
| :--- | :--- |
| Dragging the scale handle by eye in the Scene view | Typing the calculated values into the Inspector |
| Forcing a square such as `X=5, Y=5` | Deriving Y from the texture ratio |
| Non-uniform scale on the parent object | Keeping the parent at `1,1,1` and scaling the child |
| Stretching a Canvas UI element | Using a Quad + Unlit material for world placement |

> [!CAUTION]
> **Watch the parent transform.** If a parent carries a non-uniform scale (e.g. `X=2, Y=1`), the child Quad still renders distorted even when its own ratio is correct. Keep the parent scale at `1, 1, 1`.

## 6. Sharpness at Distance — Mipmap & Aniso

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| Shimmering at a distance | No mipmaps | Enable `Generate Mip Maps` |
| Blurry at a distance | Mipmaps too aggressive | `Mip Map Filtering: Kaiser`, Bias `-0.5` |
| Blurry at grazing angles | Low anisotropic filtering | Raise `Aniso Level` to `4`–`8` |
| Thin line at the edges | Tiling wrap mode | Set `Wrap Mode: Clamp` |
| Aliasing when enlarged | Insufficient resolution | Re-export at higher resolution from the vector source |

## 7. Optimization Checklist

- [ ] Material uses an `Unlit/*` shader
- [ ] `Alpha Is Transparency` is enabled (no black halo)
- [ ] Quad scale matches the texture ratio exactly
- [ ] Parent object scale is `1, 1, 1`
- [ ] `Contribute Global Illumination` is disabled
- [ ] `Cast Shadows` is `Off`
- [ ] The plane sits at least `0.01` units off the wall
- [ ] Tint Color is white (no recoloring)
- [ ] Texture `Max Size` is not excessive
- [ ] **The credit line is present in the world description**
- [ ] **Prior written approval from the Lead Director has been obtained**

## 8. Troubleshooting

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| Logo looks dark | Standard shader reacts to lighting | Switch to `Unlit/Transparent` |
| Black halo around edges | `Alpha Is Transparency` off | Enable it and press `Apply` |
| Transparent area is white/black | PNG has no alpha channel | Use the transparent version of the file |
| Logo is stretched | Quad scale ratio mismatch | Recalculate with the formula in section 5 |
| Flickering against the wall | Z-fighting | Offset `0.01`–`0.02` units from the wall |
| Invisible from behind | Quad is single-sided | Duplicate the Quad rotated 180° |
| Blotches after lightmap bake | GI contribution enabled | Disable `Contribute Global Illumination` |
| Translucent sorting flips | Render queue ordering | Set `Render Queue` manually to `3001`+ |
| Quality drops after upload | Compression format | Use `High Quality` with an alpha-preserving format |

---

<div align="center">

**© 2026 VRChat 한국수어교실 (Lead Director: 이심율) — All rights reserved.**

Licensed under CC BY-NC-ND 4.0 · Prior written approval required for all use.

[README](../README.md) · [BRANDING](../BRANDING.md) · [LICENSE](../LICENSE)

</div>
