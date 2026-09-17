---
title: "PortSwigger Web Security Academy - Unprotected admin functionality"
date: 2026-09-17
tags: [PortSwigger, WebSecurity, AccessControl, 学習ログ]
category: Access Control
difficulty: Apprentice
status: Solved
---

# Unprotected admin functionality

> PortSwigger Web Security Academy / Access Control のラボ writeup。
> カテゴリ: Broken Access Control（垂直権限昇格）

## TL;DR
`robots.txt` に記載されていた管理パネルのパス `/administrator-panel` に、UIリンクを経由せず
URLを直接指定してアクセス。サーバー側で認可チェックが無かったため、一般利用者の立場のまま
管理者専用のユーザー削除機能を実行できてしまう、という垂直権限昇格の脆弱性を確認した。

## 対象ラボ
- Lab: Unprotected admin functionality
- URL: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality
- 脆弱性カテゴリ: Broken Access Control / 垂直権限昇格
- ゴール: 管理パネルにアクセスし、対象ユーザーを削除する

## 背景知識（なぜ成立するのか）
- **保護されていない機能**: 管理機能へのリンクを管理者の画面にしか表示していないだけで、
  URL自体にアクセス制御をかけていない状態。「リンクを隠す＝守っている」ではなく、
  URLを直接叩けば誰でも到達できてしまう。
- **robots.txt の落とし穴**: 本来は検索エンジンのクローラーに巡回してほしくないパスを
  伝えるためのファイル。しかし誰でも閲覧でき、人間のアクセスを止める力は無いため、
  `Disallow` に書いた管理パスが逆に「隠したい場所の地図」として攻撃者に漏れてしまう。

## 攻略手順
### 1. 偵察 (Recon)
サイトルートの `/robots.txt` を確認したところ、管理パネルのパスが記載されていた。
```
User-agent: *
Disallow: /administrator-panel
```

### 2. 直接アクセス
UI上にリンクは存在しないが、判明したパスをURLに直接指定してアクセスした。
- アクセスしたURL: `https://<lab-id>.web-security-academy.net/administrator-panel`
- 結果: 認可チェックが無く、管理パネル（ユーザー管理画面）が表示された

### 3. 権限昇格の実証 (Exploit)
- 実行した操作: 管理パネル上のユーザー削除機能で対象ユーザーを削除
- 結果: ラボが Solved になり、垂直権限昇格が成立することを確認

## 根本原因
脆弱性の本質は「削除できたこと」ではなく、`/administrator-panel` 系のエンドポイントに
**サーバー側の認可（authorization）チェックが無かったこと**。
認可をUI層のリンク表示に依存させ、リクエストごとの権限検証を行っていない点が原因。
robots.txt はあくまで発見のきっかけであり、仮に記載が無くてもパスの推測やブルートフォースで
到達可能だった。

## 対策 (Remediation)
- 管理系エンドポイントで、リクエストごとにセッションの権限（adminロール）を検証し、
  権限が無ければ 403 等で拒否する。
- 認可の判断を必ずサーバー側で行い、UIのリンク有無に依存しない。
- robots.txt を防御手段として使わない（機密パスを記載しない）。

## 学び / 感想
- 「リンクが無い＝守られている」という思い込みが脆弱性の温床になると体感できた。
- robots.txt は診断・偵察の初手として今も有効。防御ではなく情報源として見る視点を得た。
- OWASP Top 10 で1位の Broken Access Control の、最も基本的な形を実際に手を動かして理解できた。

## 参考リンク
- [OWASP: A01:2021 – Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [PortSwigger: Access control vulnerabilities](https://portswigger.net/web-security/access-control)
