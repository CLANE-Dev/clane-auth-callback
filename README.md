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

## 使い方（製品ごと）— 製品別の「クエリ無し固定ページ」を使う（推奨）

> **重要（実地で判明）:** Supabase の `redirect_to` に **クエリ文字列（`?app=...`）を付けると
> 許可リスト照合が通らず**、Site URL（既定 `localhost:3000`）にフォールバックする。
> 一方、**クエリ無しの URL（カスタムスキーム `clane-xxx://auth` や素の https パス）は確実に通る。**
> そこで製品ごとに **クエリ不要の固定パスのページ**を用意し、`redirect_to` はクエリ無しにする。

| 製品 | スキーム | redirect_to に渡す URL（クエリ無し・推奨） |
|---|---|---|
| Form Auto Runner | `clane-form-ar` | `https://clane-dev.github.io/clane-auth-callback/form-ar.html` |
| awpw | `clane-awpw` | （必要時 `awpw.html` を追加。当面は `clane-awpw://auth` 直行のまま） |

各ページは固定スキームへ `#access_token=...` を引き継いで転送する（`form-ar.html` は
`clane-form-ar://auth` 固定）。新製品を足すときは `<product>.html` を1枚追加する。

`auth-callback.html`（`?app=` で切替える汎用版）も残しているが、上記の Supabase 挙動のため
**本番の redirect_to にはクエリ無しの製品別ページを使うこと**。

## 設定手順（製品側）

1. **Supabase**: Authentication → URL Configuration → **Redirect URLs** に
   `https://clane-dev.github.io/clane-auth-callback/form-ar.html` を追加（**クエリは付けない**）。
   カスタムスキーム `clane-form-ar://auth` も fallback として残す。
2. **製品の設定**（Form Auto Runner の `account-config.js`）:
   `authRedirect: 'https://clane-dev.github.io/clane-auth-callback/form-ar.html'`

## 正準ソース

このページの**正準ソースは別リポジトリ `CLANE-Dev/clane-account` の `web/auth-callback.html`**。
本リポジトリは GitHub Pages 配信の実体（ミラー）。内容を変更するときは両者を揃えること
（詳細は clane-account の DECISIONS.md）。
