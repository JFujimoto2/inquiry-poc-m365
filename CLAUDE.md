# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

kintoneからMicrosoft Power Platformへ移行する、施設予約管理システムのPoCプロジェクト。
対象施設は笠間・木更津の2拠点。現場スタッフが自分たちで運用・修正できる環境を目指す。

## フェーズ構成

- **Phase 1（現在）**: Power Platform（Power Apps + Power Automate + Dataverse）で問い合わせ〜見積〜予約確定の業務を自動化
- **Phase 2**: Power Pagesで顧客向けマイページを追加
- **Phase 3**: Rails（Render cron）経由でaiPass（PMS）とDataverseを定期同期

## 技術スタック

- **データストア**: Dataverse（テーブル間リレーションあり）
- **業務画面**: Power Apps（Canvas App）
- **自動化**: Power Automate（見積PDF生成、メール送信、Teams通知）
- **帳票**: Wordテンプレート → PDF変換
- **メール**: Outlook（M365テナント）
- **ライセンス**: デジ推=E5、現場=E3（Dataverse利用にはPower Apps Premium別途必要）

## リポジトリ構成

現時点ではコード成果物はなく、設計ドキュメントのみ。
- `docs/` — Phase 1の業務フロー、システム構成図、構築作業リスト、Power Platform構成まとめ

## コミット時の注意事項

- `docs/` 配下に新規ファイルを追加・変更する場合は、コミット前に個人名・メールアドレス等の個人情報が含まれていないことを必ず確認すること
- 個人情報を含むファイル（名簿、連絡先一覧、担当者リストなど）は絶対にコミットしないこと
- 担当者名は「担当者さん」「開発者さん」等の一般名称に置き換えること
