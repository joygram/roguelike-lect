# 32 — Unity 씬 · 게임 흐름 · 씬 전환 · 프리팹

**역할**: 씬·전환·Build·씬별 프리팹 내용은 **아래 표처럼 저장소 전체 흐름에 통합**되어 있습니다. 32만 열지 않아도 **README → 02 → 04 → 07** 만으로 동일 규약을 따를 수 있습니다.

| 내용 | 정본 위치 |
|------|-----------|
| 씬 다이어그램·권장 씬 목록·Build 순서·씬 전환 담당·**씬별 `_Framework`/`_Content`/`_UI` 표**·Phase별 할 일·`SceneFlow` C# 예 | [README `#scene-flow`](../README.md#scene-flow) |
| 요구·씬 표·데이터 흐름과의 연결 | [04 §2.3~2.5](04-요구사항-및-시스템설계.md) |
| 씬 **파일명**·프리팹 이름 표 | [02 §4.2](02-준비사항-체크리스트.md) |
| Phase 0 실습·완료 조건 | [07 Phase 0](07-단계별-개발-가이드.md) |
| HUD·버튼→씬 전환 | [10](10-UI-HUD-디자인-및-구현-가이드.md) |
| 스테이지 씬·stageId·맵 SO | [README #map-asset-pipeline](../README.md#map-asset-pipeline) · [28](28-기획서-맵구성-에셋-구현-연동.md) |
| 빌드 최종 확인 | [19](19-분야별-분리-통합-및-제품출시-가이드.md) |

**과제·PDF**: [README.md `#scene-flow` 절](../README.md#scene-flow)만 인쇄하면 됩니다.

**폴더 요약**: 씬 전환·런 상태 → **`_Framework/`**, 스테이지·맵·적 → **`_Content/`**, 메뉴·HUD → **`_UI/`** — [README #folder-layers](../README.md#folder-layers).

---

`<a id="step-gate"></a>`

## 단계 연결 · 준비 확인 (이 문서)

**README 순서**: Phase 0 전제. **필수**: [README #scene-flow](../README.md#scene-flow) + [02 §4.2](02-준비사항-체크리스트.md). **다음**: [07 Phase 0](07-단계별-개발-가이드.md).
