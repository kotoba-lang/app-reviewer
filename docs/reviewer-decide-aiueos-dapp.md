# app-reviewer（判定）— itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-reviewer はアプリ公開レビュー —— 開発者の提出を受け、自動チェックと
人手レビューで可否を決める（capabilities の ontology / taxonomy 付き）。

## このスライス: レビュー判定コア

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/reviewer_decide.cljk` — severity-ok（**critical / high / medium / low の 4 段**）/ auto-fail（**critical か high の失敗で auto_failed** = ブロッキング）/ decision-code（**emergency-revoke > reject > request-changes > approve** の優先順） | **実装済み・実測** |
| 記述子 | `aiueos/reviewer-decide.edn` | **実装済み** |
| parity | `design/reviewer/decide_parity_test.cljk` — 正本（design/review-process.md）の重大度・自動ゲート・決定ポリシーと一致 | **実装済み・実測** |

実測: `amu test aiueos/reviewer_decide.cljk` → **18/18**（jvm-kir / js / wasm）。
parity → 11 assertions / 0 failures。

## 正本の正確な規則（guest はこれに追従 —— review-process.md の明文）

- **重大度**: critical（exploit / データ持ち出し / malware 的挙動）/ high
  （重大な auth / risk / policy 問題）/ medium（重大バグ・誤認挙動）/ low。
- **自動チェック**: critical か high の失敗があれば status は `auto_failed`。
- **決定ポリシー**（guest の優先順と同一）:
  1. **Emergency revoke**: 公開後の critical 事故
  2. **Reject**: 重大なポリシー・セキュリティ違反
  3. **Request changes**: 修正可能な finding が残る（未解決の critical/high も
     approve させない）
  4. **Approve**: 未解決の critical/high が無い

## 未実装境界

- 提出物の検証（digest / 署名 / CVE / 静的ルール / sandbox smoke）の判定 ——
  自動チェックの各項目。policy 宣言と実挙動の diff も同様。
- capabilities の ontology / taxonomy（jsonld）の判定化。
- mon（DNA）化、aiueos session 載せ（fleet 共通）。
