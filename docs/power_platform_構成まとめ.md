# 新システム予約管理システム：Power Platform構成まとめ

## 採用方針

kintoneからPower Platformへ移行する。現場が自分たちで運用・修正できる環境を優先する。

## 前提条件

| | ライセンス | 役割 |
|---|---|---|
| デジ推 | M365 E5 | 構築・保守 |
| 現場 | M365 E3（※要確認） | 運用 |

- メール送信：M365テナントのOutlookを使用（ドメイン問題なし）
- Phase 1はPower Platformのみで完結する構成とする

---

## システム構成（Phase 1）

```
Dataverse（データストア）
    ↓
Power Apps（業務画面）／Power Automate（自動化）
```

---

## 役割分担（Phase 1）

| 役割 | 技術 | 担当 |
|------|------|------|
| 問い合わせ・業務画面 | Power Apps | 現場が運用 |
| 自動化フロー（メール送信等） | Power Automate | 現場が確認・修正 |
| 見積PDF生成 | Power Automate + Wordテンプレート | 開発者さんが初期構築 |
| 空き状況カレンダー | Power Apps内（手動入力ベース） | 開発者さんが初期構築 |
| データストア | Dataverse | Premiumライセンス別途必要 |

---

## ライセンスコスト試算

| 機能 | デジ推（E5） | 現場（E3） |
|------|------|------|
| Power Apps（SharePoint接続） | ✅ 含まれる | ✅ 含まれる |
| Power Apps Premium（Dataverse接続） | ✅ E5に含まれる | ❌ **別途約2,000円/ユーザー/月** |
| Power Automate（標準コネクタ） | ✅ 含まれる | ✅ 含まれる |
| Power BI Pro | ✅ 含まれる | ❌ **含まれない** |
| Power Pages（顧客マイページ） | ❌ 別途費用（Phase 2で検討） | ❌ 別途費用 || aiPass等外部APIへの直接接続 | Premiumコネクタ必要→別途費用（Phase 3で検討） | 同左 |

### ⚠️ E3での注意点
- **Power BI Proが含まれない**ため、空き状況カレンダーはPower Apps内に組み込む形で対応する（Power BIを使わない）
- **DataverseへのアクセスにはPower Apps Premiumライセンスが必要**（E3には含まれない）。現場スタッフ分は別途購入が必要

### Power Pages（Phase 2）のコスト試算

Power Pagesは**月次ユニークユーザー数**ベースの課金モデル。認証ユーザーと匿名ユーザーで料金体系が異なる。

| ユーザー種別 | 課金単位 | 想定ユーザー |
|---|---|---|
| 認証ユーザー | 100人単位のパックで購入 | ログインして条件変更する顧客 |
| 匿名ユーザー | 500人単位のパックで購入 | 空き状況を閲覧するだけの訪問者 |

**新システムへの影響：**
- 顧客マイページ（ログインして条件変更）は**認証ユーザー**扱い
- 月の問い合わせ・予約件数が少ない規模（数十件/月）であれば、最小パックで収まる可能性が高い
- **Power Apps Premiumライセンス（約2,000円/ユーザー/月）を保有している社内ユーザーはPower Pagesのカウントから除外**されるため追加費用なし
- 具体的な金額はMicrosoftの営業担当またはパートナーに要確認

---

## kintoneとの機能対応

| kintone現行 | Power Platform代替 |
|------|------|
| Form bridge（問い合わせフォーム） | Power Apps |
| レポトン（PDF見積書生成） | Power Automate + Wordテンプレート |
| K viewer カレンダー（空き状況） | Power Apps内カレンダー |
| K mailer（メール送信） | Power Automate + Outlook |
| キントーン（データ管理） | Dataverse + Power Apps |

---

## フェーズ構成

### Phase 1：スタッフ側の業務効率化（Power Platformのみ）
- 問い合わせ〜見積〜予約確定の自動化
- 見積PDF自動生成・メール送信
- SharePointリストによるデータ管理
- 空き状況カレンダー（手動入力ベース）

### Phase 2：顧客向け機能
- 顧客マイページ（条件変更・確認）
- Power Pagesの導入検討（別途費用が発生するため要判断）


## システム構成（Phase 3以降）

```
aiPass（PMS）
    ↓ 定期取得（Render cron）
Rails（同期処理のみ・軽量）
    ↓ 整形して書き込み
Dataverse（データストア）
    ↓
Power Apps（業務画面）／Power Automate（自動化）
```

### Phase 3：PMS連携
- aiPass APIとの定期同期
- Rails（Render cron）でSharePointへ自動書き込み
- 空き状況の自動反映

---

## データストア選定

### SharePoint リスト
- ✅ E3・E5ともに含まれる（追加費用なし）
- ✅ 現場がブラウザから直接データを見て編集しやすい
- ❌ テーブル間のリレーションが弱い
- ❌ 5,000件制限の壁がある

### Dataverse
- ✅ テーブル間のリレーションがしっかり組める
- ✅ Power Appsとの相性が最も良い（動作が速い）
- ✅ 入力バリデーション・ビジネスロジックをDB側に持たせられる
- ❌ E5に含まれない（Power Apps Premiumライセンス別途必要：約2,000円/ユーザー/月）

### 方針
**DataverseをPhase 1から採用する。**

kintoneの既存データを移行することを考えると、テーブル間のリレーションがしっかり組めるDataverseの方が適している。Power Apps Premiumライセンスが別途必要になるが、現場スタッフ数名分（担当者等）のコストは許容範囲と判断する。

| ユーザー数 | 追加コスト目安 |
|---|---|
| 2名 | 約4,000円/月 |
| 5名 | 約10,000円/月 |

---

## 技術選定の経緯

当初Rails自作も検討したが、以下の理由でPower Platformを採用した。

| | Power Platform | Rails自作 |
|---|---|---|
| 構築速度 | 速い | 遅い |
| 月額コスト | E3/E5範囲内（追加ほぼなし） | Renderサーバー代のみ |
| 柔軟性 | 低い（ローコードの限界） | 高い |
| 現場保守 | しやすい | 開発者さん依存 |

→ **現場運用を優先するためPower Platformを採用。** aiPass同期はPhase 3でRailsを軽量に追加する。