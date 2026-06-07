# clane-auth-callback

CLANE アカウント認証（Supabase + Google ログイン）の **OAuth 戻り中継ページ**を GitHub Pages で配信するための公開リポジトリ。

CLANE 製品（Electron デスクトップアプリ）は、外部ブラウザでの Google ログイン後にカスタム URL
スキームでアプリへ戻る。スキーム直行（`clane-xxx://auth#...`）だと、認証後のブラウザタブが
開いたまま残りユーザーが戸惑う。そこで Supabase の戻り先（`redirect_to`）をこの HTTPS ページにし、
「ログインが完了しました。このタブは閉じてかまいません」を表示しつつアプリのカスタムスキームへ
受け渡す。

## 配信 URL（GitHub Pages）

```
https://clane-dev.github.io/clane-auth-callback/auth-callback.html
```

## 使い方（製品ごと）

製品は `redirect_to` に `?app=<スキーム>` を付けて指定する。ページはそのスキームへ
`#access_token=...` を引き継いで転送する。

| 製品 | スキーム | redirect_to に渡す URL |
|---|---|---|
| awpw | `clane-awpw` | `https://clane-dev.github.io/clane-auth-callback/auth-callback.html?app=clane-awpw` |
| Form Auto Runner | `clane-form-ar` | `https://clane-dev.github.io/clane-auth-callback/auth-callback.html?app=clane-form-ar` |

`?app=` 未指定・未知のスキームは `clane-awpw` にフォールバック。許可スキームは
`auth-callback.html` 内の `ALLOWED` 配列で管理（任意スキームへの転送を防ぐため）。新製品を
追加するときは `ALLOWED` に追記し、この表も更新する。

## 設定手順（製品側）

1. **Supabase**: Authentication → URL Configuration → Redirect URLs に
   `https://clane-dev.github.io/clane-auth-callback/auth-callback.html` を追加
   （`?app=` 付きクエリは同一ベース URL として許可される。カスタムスキーム `clane-xxx://auth` も併せて残す）。
2. **製品の設定**（例: Form Auto Runner の `account-config.js`）:
   `authRedirect: 'https://clane-dev.github.io/clane-auth-callback/auth-callback.html?app=clane-form-ar'`

## 正準ソース

このページの**正準ソースは別リポジトリ `CLANE-Dev/clane-account` の `web/auth-callback.html`**。
本リポジトリは GitHub Pages 配信の実体（ミラー）。内容を変更するときは両者を揃えること
（詳細は clane-account の DECISIONS.md）。
