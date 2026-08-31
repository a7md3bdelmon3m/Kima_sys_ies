# DECISION LOG

## ملاحظة أمانة قبل أي شيء آخر

هذا الملف **غير موجود في الحزم التي رُفعت لهذه الجلسة** (`KIMA_EIS_V3_ADR034_2026-08-31.zip`
و`30_WEB_APP.zip`)، رغم أن `00_CONTROL/PROJECT_STATE.md` يشير إليه في عشرات
المواضع كسجل القرارات المركزي منذ ADR-001. هذا الملف الحالي **ليس** إعادة
بناء لذلك التاريخ — لا تملك هذه الجلسة محتواه الأصلي، ولا تخترعه.

إن كان لديك نسخة سابقة من `DECISION_LOG.md` (محليًا أو في حزمة لم تُرفع
بعد)، يجب دمج هذا الملف معها يدويًا — لا استبدالها — لتفادي فقدان سجل
ADR-001 حتى ADR-034.

---

## ADR-035 (2026-08-31) — R3-002 context-bar-as-filter, REAL-DEVICE CONFIRMED

**Session**: User's own Android Chrome device, app served at
`http://127.0.0.1:18625` (local hosted mode — a real HTTP origin, not
`file://`).

**What was tested**: R3-002 — the persistent context bar (Plant/Area/Unit/
System breadcrumb) as a clickable navigational filter over the entity
explorer list, per `01_PRODUCT/MOBILE_UX_REQUIREMENTS.md` and
`00_CONTROL/tasks/R3-002-TASK_RESULT.md`'s original acceptance criteria.

**Method**: Three entities created live via the real UI, each with a
hierarchical `context` value:
- `PUMP-101-A` — context: unset (no hierarchy)
- `VALVE-201-B` — context: `PlantA/AreaB/Unit1`
- `LINE-301-C` — context: `PlantA/AreaB/Unit2`

Steps executed and screenshot-confirmed:
1. Selected `VALVE-201-B` → breadcrumb `PlantA ‹ AreaB ‹ Unit1` rendered
   correctly above the header stats line.
2. Selected `LINE-301-C` → breadcrumb updated to `PlantA ‹ AreaB ‹ Unit2`,
   confirming the bar tracks the currently-selected entity's own context
   (not a stale/cached value).
3. Tapped the `AreaB` segment specifically (middle segment, not `PlantA`
   or `Unit1`) → segment rendered with an active-state underline.
4. Explorer list updated to show a `مُصفّى حسب السياق: PlantA › AreaB`
   indicator with an explicit `إزالة التصفية` (clear filter) control, and
   the list narrowed from 3 entities to exactly 2 (`LINE-301-C`,
   `VALVE-201-B`) — correctly excluding `PUMP-101-A` (no hierarchical
   context, so never a match).

**Result**: All three acceptance criteria from the original R3-002 task
confirmed on a real device for the first time:
- Breadcrumb correctly parses and renders a hierarchical `context` value.
- Breadcrumb display is independent across different selected entities
  (re-renders per-selection, not stuck on first render).
- Clicking a breadcrumb segment narrows the explorer list correctly,
  with a visible, dismissible filter-active indicator — not a silent or
  ambiguous state change.

**Gap register**: M-17 → CLOSED, real-device-verified (previously OPEN;
R3-001's headless-Chromium harness had only confirmed the breadcrumb
*renders*, not the click-to-filter interaction itself — see
`00_CONTROL/tasks/R3-002-TASK_RESULT.md`'s prior PARTIAL status).

**Scope note**: This confirms the `PlantA/AreaB` two-level narrowing case
on a real device. Not yet exercised on this device: filtering by the
full three-level path (`PlantA/AreaB/Unit1` exactly, narrowing to one
entity), the `PlantA`-only (single-segment) filter, or the clear-filter
control's actual click behavior (only its presence was confirmed). These
remain reasonable follow-ups but are not blocking — the core
click-to-filter mechanism this ADR set out to confirm is now evidenced.

**Not verified by this ADR**: Any other Gate D item. See
`00_CONTROL/PROJECT_STATE.md`'s "Next task" section for the remaining
real-device queue (R2-001/R2-002/R2-003 CSV/Excel import on-device,
T012/T013 file-picker and install-cycle paths, H-03 mutation ops — still
no UI wiring at all).
