# 301 リダイレクト管理（サービス再編 2026）

サーバの `.htaccess` は **WordPress / セキュリティプラグイン / GeoIP** が管理しており、
**デプロイ（rsync）では `.htaccess` を除外**している（`deploy.yml` / `deploy-production.yml` の `--exclude='.htaccess'`）。
そのため、リダイレクトは **各サーバの `.htaccess` に手動で統合**する。リポジトリからは自動反映されない。

## リダイレクトブロック（共通）

```apache
# BEGIN CLANE Service Restructure 301
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule ^service-dx/?$        /service-ai-consulting/ [R=301,L]
RewriteRule ^service-system/?$    /service-newbiz/        [R=301,L]
RewriteRule ^service-subsidy/?$   /service-newbiz/        [R=301,L]
RewriteRule ^service-lab/?$       /service/               [R=301,L]
RewriteRule ^service-erp-v1/?$    /service-erp/           [R=301,L]
</IfModule>
# END CLANE Service Restructure 301
```

## 適用場所（環境で異なる）

### 本番（clane.co.jp）

デプロイ先 `/home/libertyclan/clane.co.jp/public_html/` の **既存 `.htaccess` に統合**する。
`#End Really Simple Security` の直後、`# BEGIN WordPress` の **直前** に上記ブロックを挿入。
WordPress/セキュリティ/GeoIP 等の既存設定は**一切変更しない**。
（統合済みファイル: デスクトップ `htaccess_production_NEW.txt`）

### ステージング（clane.check-demo.site）

デプロイ先はサブディレクトリ `/home/clane000/check-demo.site/public_html/clane.check-demo.site/`。
**親 `public_html/.htaccess`（BASIC認証・IP許可・親WP）は触らない。**
デプロイ先サブディレクトリに **リダイレクトのみの `.htaccess` を新規設置**する。
（ファイル: デスクトップ `htaccess_staging_NEW.txt`）
※ 親のBASIC認証・IP許可は継承されるため影響なし。サブディレクトリに mod_rewrite を置くと親のWPリライトはこのディレクトリでは継承されないが、静的サイト（実ファイルを直接配信）なので問題ない。

## 旧URL → 新URL 対応表

| 旧URL               | 新URL                     | 理由                                |
| ------------------- | ------------------------- | ----------------------------------- |
| `/service-dx/`      | `/service-ai-consulting/` | DXコンサルをAIコンサルの土台に転用  |
| `/service-system/`  | `/service-newbiz/`        | MVP・アジャイルを新規事業開発に統合 |
| `/service-subsidy/` | `/service-newbiz/`        | 補助金を新規事業開発ページに内包    |
| `/service-lab/`     | `/service/`               | ラボ型開発を廃止                    |
| `/service-erp-v1/`  | `/service-erp/`           | ERP旧版の廃止                       |

## 注意

- **タイミング**: リダイレクト先の新ページが公開された後（同一リリース）に有効化する。先に有効化すると現行の `/service-dx/` 等が404転送になる。
- **物理ファイル**: rsync は `--delete` なしのため、リポジトリから削除した旧ディレクトリ（service-dx 等）はサーバ上に残る。301で隠れるが、いずれサーバ上から手動削除するのが望ましい。
