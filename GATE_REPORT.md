# GATE REPORT — 2026-07-19 (Opus 모드 O)

## ★★ 인페인팅 서비스화 3계층 — 설계 확정·프로토타입 착수 (2026-07-19)

서비스화 전제(타 사용자 대상)로 인페인팅을 **무료 기본 + BYOK 2단(+실패 폴백)** 재설계. 조사→설계 완료, 사용자 승인(조건 10건 반영), **1페이지 프로토타입 단계**.

- **설계 스펙**: `docs/superpowers/specs/2026-07-19-inpainting-service-tiers-design.md` (MurderLab repo, commit `7c420ad`).
- **3계층**: `BasicFillProvider`(색블록·기본·무료) < `MiganOnnxProvider`(MI-GAN, 토글·무료) < `CloudEraseProvider`(ClipDrop, BYOK). 각 단계 실패 시 아래로 **라벨 붙인 명시적 폴백**(조용한 폴백 금지).
- **T1=MI-GAN** (MIT, ONNX 28MB, WebGPU→WASM). big-lama·MAT는 비상업 라이선스로 탈락. Places2 잔여 리스크 감수(Picsart 명시 MIT 부여로 방어), 교체=설정 수준(문제 시 즉시 색블록 강등), 배포물 크레딧 표기.
- **T2=ClipDrop Cleanup**(erase형, 저환각). generative fill(Firefly/OpenAI/Imagen) 배제=환각 위험. **무상태 프록시**(키 헤더 전용·서버 미저장·로그 위생 가드), BYOK 에러 크게 표면화(키무효/크레딧소진/업스트림 구분). Stability Erase=2순위 기록.
- **기존 불변식 재사용**: `InpaintProvider`·팩토리·`originalCanvas` 정합·배치 진행률·프로바이더 라벨 — **호출부 무변경**.
- **판정 대기(사용자 게이트)**: 프로토타입 = **실물 설정집 페이지(격자 배경)**로 현행 색블록 vs MI-GAN 전후 비교(합성 검증 금지). 판정 기준 = 격자선 관통 연속성 / 번짐·얼룩 아티팩트 / 색블록 대비 육안 우위 + 수치(다운로드 용량·시간, 페이지당 추론 시간, 폴백 발생). 스파이크는 브랜치, 눈확인 통과 후 본 구현.

> **채널 무결성 메모(2026-07-19)**: ml-gate push는 정상(로컬==원격). 게이트가 stale `056703a`(GATE_REPORT 부재 시점 커밋)를 읽는 건 push 문제 아님 → 게이트 raw URL이 커밋 고정 permalink이거나 엉뚱한 경로일 가능성(사용자 확인 필요). 이후 "ml-gate 갱신됨" 보고 완료 정의 = `git ls-remote origin main` 해시 동반.

## ★★ R2 인라인 colorRuns — 눈확인 통과·main 병합 완료 (2026-07-19)

색 강조를 별도 span 조각(R1) → **문단 안 인라인 colorRuns** 로 전면 전환 완료.
**최종 눈확인 통과 → `feat/r2-inline-colorruns` → main `--no-ff` 병합**(merge `3a3db98`, **329 그린 + 빌드**),
origin push, 브랜치 삭제(로컬+원격). **서버 = main 프로덕션 빌드(:3000) 가동 중.**

- **눈확인 판정**(사용자 게이트): 인라인 즉시 색 변경 O / 한 문단 다색 공존 O / **커닝 이음새 자연**
  (확대 캡처 — 색 경계 겹침·벌어짐 없음 → **클립 렌더 하드닝 불필요, 현 스택드 프래그먼트 렌더 유지** 확정) /
  내보내기 색 반영 O(실제 PNG). 관찰(cosmetic, 미수정): 영역 드래그 중 색이 한 박자 늦게 따라옴(최종 위치 정상, USAGE_ROUND 기록).
- **후속 후보(조용한 소실 아님)**: notices UI 패널(편집/번역 경로엔 개수 명시 확인 다이얼로그 존재).

설계·Phase 계획 = `plans/2026-07-18-r2-implementation-design.md`(Claude+Sol 대조 → Sol 마스터 스펙 채택). 상세 결정 = DECISIONS 2026-07-19.

- **Phase 0 ✅** 설계+대조+합의(`1a69d4c`).
- **Phase 1 ✅** 스키마(ColorRun/colorRuns) + `colorRuns.ts`(normalize/runFragments) + wordBox 다줄 + 기본 렌더(에디터+내보내기) + cloneRegions. 가드 `colorruns.spec` 11. **309 그린 + 빌드**(`d160602`). R1 잠정 유지.
- **Phase 2 🔵(코어 완료)** setColorRun 계약 + setRegionText 재매핑(오착색<색벗김) + 번역 clear + colorRunNotices. 가드 `colorruns-edit`(7)+`colorruns-store`(6). **322 그린 + 빌드**(`70b5eba`).
  - **번역 clear 도달성 확인**(사용자 요청): "자동 번역"·카드 번역이 기번역 영역을 재번역(전면교체)해 색 강조를 지우는 경로 = **사용자 도달 가능** 확인 → 재번역 전 지워질 색 강조 **개수 명시 확인 다이얼로그** 추가(양 경로), 취소=보존. 가드 `translate-colorrun-warning`(2). **324 그린 + 빌드**(`25b332c`).
  - **남은 하위**: 클립 렌더 하드닝 + 에디터↔내보내기 픽셀 패리티 + notices UI 패널 + ingress 정규화 검증.
- **Phase 3 ✅(331 그린 + 빌드, `d7bb603`)** 로드 경계 span→colorRun 마이그레이션(`migrateColorSpans`, 텍스트 유일-일치 추론) + persistent `conversionHolds`(WorkSession 왕복) + 앰버 보류 배너(`ColorSpanHoldBanner`, 에디터·카드) 수동 삭제. 애매/무매치는 조용히 버리지 않고 보류(사전 원칙). 가드 `colorruns-migration`(7). scale-stress 를 colorRuns 로 전환. **R1 span 생성 UI 는 잠정 공존**(로드 시에만 변환).
- **Phase 4 ✅(329 그린 + 빌드)** UX 컷오버 + R1 전면 제거 완료. 3 컷오버(각 전체 회귀 그린 후 진행):
  - **컷오버 1**(`60aea2a` 다음, 329): TextStylePanel `setColorRun`(조각 없이 문단 안 즉시 색 변경 + "색 제거" 버튼=base색 적용) + RegionListPanel "다색 N" 뱃지(span 행/토글 삭제). 구 UI 스펙 재작성(partial-color-span→colorruns-ux, colorspan-list-collapse→colorruns-list-badge).
  - **컷오버 2**(328): store `addColorSpan`·`Region.isColorSpan`·`textNodeRegistry.ts` 삭제, mask `isInpaintBackground` 제거(전 region 배경, locked 만 제외), useOcr 경고 colorRuns 총수. 구 span 스펙 5종 colorRuns 로 승계/구조적 소멸 근거 DECISIONS 기록.
  - **컷오버 3**(329): 로드 경계 colorRuns 정규화(malformed→canonical) + DECISIONS.
  - **미착수(후속 후보, 조용한 소실 아님)**: 렌더 클립 하드닝(아래 커닝 드리프트 눈확인 후 판단), notices UI 패널(편집/번역 경로엔 개수 명시 다이얼로그 존재).

## R2 눈확인 체크리스트 (통과 기록, 2026-07-19)

> ✅ 전 항목 통과 → main 병합 완료. 아래는 판정 근거 기록.

1. **인라인 색 적용(조각 없음)**: 영역 선택 → 편집칸 번역문에서 단어 드래그 → 색 선택(스와치/픽커) → "선택 단어 색 강조". 새 조각/드래그 없이 그 단어만 색이 바뀌고 문단 흐름(줄바꿈·정렬) 그대로인가.
2. **색 변경**: 이미 색 있는 단어를 다시 다른 색으로 강조 → 즉시 새 색으로 교체되는가(중복/잔색 없이).
3. **색 제거**: 색 있는 단어 선택 → "색 제거" → 기본색으로 돌아가고 나머지 색 구간은 보존되는가.
4. **★커닝 드리프트(클립 렌더 하드닝 판정용)**: 색 글자와 주변 검정 글자의 **간격·정렬이 자연스러운지** — 색 경계에서 글자가 겹치거나(overlap) 벌어지는(gap) 현상이 없는지. 확대해서 색 단어 앞뒤 이음새를 본다. 드리프트가 보이면 = 클립 기반 풀텍스트 클론 렌더로 하드닝 필요(합의안 §1). 자연스러우면 = 현 스택드 프래그먼트 유지.
5. **목록 뱃지**: RegionList 부모 행에 "다색 N"(색 구간 수) 뱃지가 뜨고, 별도 span 행/접기 토글이 없는가.
6. **내보내기 일치**: 내보낸 PNG/PDF 의 색 강조가 화면과 동일 위치·색인가(에디터↔내보내기 동형).
7. **편집 재매핑**: 색 단어 뒤/앞 텍스트를 편집 → 색이 엉뚱한 글자로 옮겨가지 않는가(어긋나면 그 색만 벗겨지고 알림). 번역문 앞부분 삽입 시 색 구간이 따라 밀리는가.
8. **재번역/재-OCR 경고**: 색 있는 영역 재번역("자동 번역")·재OCR 시 "색 강조 N곳 사라짐" 개수 명시 확인 다이얼로그가 뜨고 취소=보존인가.
9. **마이그레이션 보류 배너**: 구 R1 span 세션 로드 시 자동 변환 + 애매/무매치는 앰버 배너("색 강조 변환 검토 N개")로 표면화되는가.
10. **인페인트**: 색 강조 있는 영역 전체 재생성 → 색 단어 아래 원문 픽셀도 함께 지워져 원문 비침이 없는가.

> 아래는 R2 병합 이전(main `da1bbb6`) 밤샘 배치 기록 — 현재 main 은 R2 병합본 `3a3db98`.

---

# GATE REPORT — 2026-07-18 밤샘 auto 배치 (Opus 모드 O)

**상태**: main = `91c2c4f`(③·(a) 병합 포함, 눈확인 브랜치 전부 소진), origin 동기, 병합 후 회귀 그린(298 통과)+빌드. **서버 = main 프로덕션 빌드(:3000) 가동 중 = "ㄱ 준비됨"**.

## 아침 판정 완료 (2026-07-18 아침)
1. ✅ 자율 병합 임시 브랜치 4개 삭제(-d, main 완전 병합 확인): session-type-routing-gate·vision-surface-processing-errors·legacy-roletier-normalize·scale-stress-harness.
2. ✅ ③ `feat/colorspan-list-collapse`(B+C) 눈확인 통과 → `--no-ff` 병합(RegionListPanel roleTier `!= null` 자동 해소·검증) → 회귀 294 그린 → push → 브랜치 삭제.
3. ✅ (a) `fix/new-project-reset-confirm` 눈확인 통과 → e2e 로 관찰 두 갈림 판정(Test A: 취소는 store 무해 / Test B: 재진입 re-hydrate 는 main 에도 있던 기존 동작) → `--no-ff` 병합 → 회귀 298 그린 → push → 브랜치 삭제. 가드 `tests/a-cancel-and-rehydrate.spec.ts`.
4. 카드 브랜치 2개(`fix/card-batch-autosave`·`fix/card-open-correct-page`) 계속 보류(카드 실파일 확보 후 눈확인).
- 관찰 기록(수정 안 함, `USAGE_ROUND_2026-07-14.md`): (i) "번역된 영역 11/10" cosmetic(span 이 분자만, parentId 파생배제 때 일괄) (ii) **re-hydrate 미저장 소실 = 완주 지뢰 후보**(dirty 세션 재진입 시 IndexedDB 저장본이 in-memory 덮음, 기존 동작, 완주 빈도 관찰 후 판단).

## 템플릿 리뷰 MAJOR 3건 — R2 병합 후 재판정 (2026-07-19)
- **#2 span→템플릿 payload 오염** → **무효·닫음**: 오염원 = 별도 span region(isColorSpan, 부모 tier 상속)이 `buildTemplatePayload`/`deriveSizeSet` 에서 phantom 멤버로 카운트되던 것. R2 로 span region 이 소멸(색 강조 = 부모 인라인 colorRuns)하여 파생 대상은 실 콘텐츠 region 뿐 → counts/bodyIndex 오염원 자체가 없음. `!isColorSpan` 필터 불필요(구조적 해소).
- **#3 span tier stranding** → **무효·닫음**: 고립될 별도 조각(span)이 없음. 색 강조는 부모 region 의 colorRuns 이고 렌더 프래그먼트가 부모 `fontSettings`(size/family/weight)를 읽어 티어 변경을 자동 추종(fill 만 강조색). 가드 = `colorruns-ux` "티어 크기/글씨체 변경 → colorRuns 보존, 부모 타이포 자동 추종".
- **#4 busy 중 저장** → **해결·닫음**(2026-07-19, 자율 병합 `e14309a`): SaveControls `disabled` 조건에 `isBusy` 추가 — 배치 OCR/번역/재생성 진행 중 중간저장 차단(중간 스냅샷이 정상 저장본 덮는 무결성 사고 방지) + busy 중 안내 문구 + 해제 후 정상 저장. busy-lock 패턴(장시간 버튼·undo 잠금 전례) 확장이라 눈확인 면제. 가드 `tests/save-busy-lock.spec.ts` 2건. **331 그린 + 빌드.**

## 완료 (자율 병합 + push — 검증선 통과, 브랜치는 삭제됨)
1. **#3 (b)+(c) 세션 경계** → main: openSession 이 type 으로 라우팅(카드=/cards) + 저장 직전 크로스모드 type 게이트(CrossModeSaveError 표면화). `tests/session-boundary.spec.ts` 4건.
2. **Vision wipe (#5 부류·문서 모드)** → main: Vision 200+`responses[].error` 를 던져 페이지 wipe 방지. `OcrProcessingError` 공통화. `tests/vision-adapter.spec.ts` +2.
3. **레거시 roleTier 손상(Sol 리뷰 #5)** → main: 로드 시 roleTier=null 정규화 + 필터 `!= null` 방어(setTierFont(undefined) 전 페이지 폰트 덮어씀 차단). `tests/legacy-roletier.spec.ts`.
4. **스케일 스트레스 하네스** → main: 40p×25+span60 전 파이프라인 불변식+수치. `tests/scale-stress.spec.ts`.

## 보류 (눈확인 게이트 — 병합 금지, 판정 후 --no-ff)
- **`fix/new-project-reset-confirm`** (신규, 이번 배치): #3 (a) — "새 프로젝트" 클릭 시 잔존 세션 확인 후 리셋(window.confirm). 285 그린. `tests/new-project-reset.spec.ts` 2건. ★첫 mount-effect 방식이 테스트 레이스 유발 → DashboardScreen 클릭 지점으로 이동해 해소(DECISIONS 상세).
- **기존 보류 3건**(2026-07-16~17, 이제 main 보다 9+ 커밋 뒤): `feat/colorspan-list-collapse`(B+C), `fix/card-batch-autosave`(#1), `fix/card-open-correct-page`(#2). 각 판정 통과 시 병합(3-way, 파일 분리라 충돌 예상 안 됨).

## 판단 요청 (아키텍처/설계 — 밤샘 단독 재설계 금지로 기록만)
- **parentId vs R2 결정**: `plans/2026-07-18-parentId-vs-R2-design.md` — 추천 **Option A(parentId)**, 뒤집는 데이터 = **OP2(한 부모 2+색 흔함)**. 완주 관찰 3포인트 매핑 완료 → 데이터 오면 즉시 결정.
- **템플릿 리뷰 잔여 MAJOR 3건**: 위 "템플릿 리뷰 MAJOR 3건" 섹션 참조(#2 payload 오염 / #3 stranding / #4 busy 저장). #2·#3 은 parentId 패키지, #4 는 writer-lock. 판정은 완주 후.
- **에러 정책(기록)**: generic provider 에러→Mock 폴백이 실데이터 덮음 / 빈 정상 OCR→기존작업 카드 wipe / export 에러 UI 미표면화.

## 다음 (아침 ㄱ 이후)
1. 보류 4브랜치 눈확인 → 병합 판정.
2. 실사용 완주(설정집 5p~끝) 재개 → OP1/OP2/OP3 관찰 로그(`USAGE_ROUND_*.md`) → parentId/R2 확정.
3. 위생 후속: ReflowControls React key 경고 원인 규명(현재 cosmetic, 모든 .map key 보유해 불명).
