---
title: "PortSwigger Web Security Academy - Unprotected admin functionality with unpredictable URL"
date: 2026-09-17
tags: [PortSwigger, WebSecurity, AccessControl, 学習ログ]
category: Access Control
difficulty: Apprentice
status: Solved
---

# Unprotected admin functionality with unpredictable URL

> PortSwigger Web Security Academy / Access Control のラボ writeup。
> カテゴリ: Broken Access Control（垂直権限昇格 / Security through obscurity）

## TL;DR
管理パネルのURLは推測しにくいランダム文字列で「隠蔽」されていたが、ロールに応じて
リンクを出し分けるフロントエンドの JavaScript にURLがベタ書きされており、権限に関係なく
全ユーザーのブラウザに配信されていた。ページのソースからURLを読み取って直接アクセスし、
管理者専用のユーザー削除機能を実行。隠蔽（security through obscurity）は防御にならないことを確認した。

## 対象ラボ
- Lab: Unprotected admin functionality with unpredictable URL
- URL: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url
- 脆弱性カテゴリ: Broken Access Control / 垂直権限昇格
- ゴール: 隠された管理パネルにアクセスし、対象ユーザーを削除する

## 背景知識（なぜ成立するのか）
- **隠蔽によるセキュリティ (Security through obscurity)**: 機密機能のURLを推測されにくく
  することで守ろうとする考え方。URL自体にアクセス制御が無ければ、場所がバレた時点で破られる。
- **フロントエンドからのURL漏洩**: `if (isAdmin)` のようにロールで表示を出し分けても、
  判定を行うJavaScript本体（＝URL文字列を含むコード）は全ユーザーのブラウザに配信される。
  一般ユーザーでもソースを読めば隠しURLを取得できてしまう。

## 攻略手順
### 1. 偵察 (Recon)
トップページのHTMLソース（またはDevTools）を確認し、管理パネルへのリンクを生成している
JavaScript を発見。ロール判定のコード内に管理パネルのURLがベタ書きされていた。
```html
<script>
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-n8sf5s');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
</script>
```

### 2. 直接アクセス
JSから読み取ったパスをURLに直接指定してアクセスした。
- アクセスしたURL: `https://<lab-id>.web-security-academy.net/admin-n8sf5s` 
- 結果: 認可チェックが無く、管理パネルが表示された

### 3. 権限昇格の実証 (Exploit)
- 実行した操作: 管理パネル上のユーザー削除機能で対象ユーザー(Carlos）を削除
- 結果: ラボが Solved になり、垂直権限昇格が成立することを確認

## 根本原因
URLを推測困難にする「隠蔽」に依存し、`/admin-n8sf5s` エンドポイントに
**サーバー側の認可チェックが無かったこと**が本質。さらに、秘密であるべきURLを
クライアント側JavaScriptに含めたことで、隠蔽そのものも成立していなかった。

## 対策 (Remediation)
- 管理系エンドポイントで、リクエストごとにセッションの権限（adminロール）を検証して拒否する。
- 秘密のURLや権限依存のロジックをフロントエンドのJavaScriptに含めない。
- アクセス制御を「URLの秘匿」に依存させない（security through obscurity をやめる）。

## 学び / 感想
- 「推測できないURL」でも、ソースを1行読むだけで漏れることを体感。隠蔽は防御ではない。
- 攻略の入口が robots.txt から「ページのJavaScript読解」に変わり、偵察の引き出しが増えた。
- 前回の Unprotected admin functionality と同じ根本原因（認可のサーバー側強制不足）が、
  別の見た目で現れることを理解できた。

## 参考リンク
- [OWASP: A01:2021 – Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [PortSwigger: Access control vulnerabilities](https://portswigger.net/web-security/access-control)
