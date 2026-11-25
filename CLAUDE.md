# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

自分の Web ページ（GitHub Pages ユーザーサイト `biuntri.github.io`）のリポジトリ。

## 現状

- 個人サイト。今は、Google Cloud の OAuth アプリ「Minecraft-server-biuntri」に登録するプライバシーポリシーを置いている。このアプリは Minecraft サーバーのバックアップを Google Drive に保存するもの。
- ポリシーは英語で、`/privacy-policy/` に置く。トップページ（`index.md`）は、アプリのホームページとして登録するためのもの。
- テストはない。
- 推測でコマンドや構成を書かない。

## 技術スタック

- Jekyll 4.4 と minima 2.5。
- Ruby 3.4。バージョンは `.ruby-version` に書き、Actions と手元で同じものを使う。
- 依存は `Gemfile` / `Gemfile.lock` で管理する。lock の PLATFORMS は `x64-mingw-ucrt` と `x86_64-linux`。
- 手元の Ruby は、Windows では RubyInstaller（DevKit 付き、`C:\Ruby34-x64`）を使う。

## コマンド

``` sh
bundle install
bundle exec jekyll serve   # プレビュー（http://localhost:4000）
bundle exec jekyll build   # 出力先は _site/（.gitignore 済み）
JEKYLL_ENV=production bundle exec jekyll build   # 本番と同じ条件でビルド（Actions もこの形）
```

`jekyll serve` では、minima 2.5.2 による Sass の DEPRECATION WARNING が出るが、エラーではない。Windows では wdm gem を勧める案内も出るが、入れなくても動く。

別の OS を lock に加えるときは `bundle lock --add-platform <platform>` を使う。

## 構成

- `_config.yml`: サイトの設定。`exclude` に `CLAUDE.md`、`README.md`、`.claude/`、`.githooks/` を入れて、公開しないようにしている。Jekyll 4.4 では既定の exclude は自動で足される。ドットで始まるファイルはもともと出力されない。公開したくないファイルを増やしたときは、ここに足す。
- `index.md`: トップページ。
- `privacy-policy.md`: プライバシーポリシー（`/privacy-policy/`）。
- `.github/workflows/pages.yml`: ビルドと公開のワークフロー。

## ポリシーを書き換えるとき

- Effective date も更新する。
- アプリ名「Minecraft-server-biuntri」は、OAuth 同意画面の名前と一致させる。

## 公開方法

GitHub Actions（`.github/workflows/pages.yml`）で GitHub Pages に公開する。

- main 向けの PR では `build` ジョブだけが走る。デプロイはしない。
- main への push で `build` と `deploy` が走り、公開される。
- 手動実行（workflow_dispatch）もできる。デプロイされるのは ref が main のときだけ。
- ジョブ名 `build` はブランチ保護の必須チェックに使う予定なので、変えない。
- リポジトリの Settings → Pages の Source を「GitHub Actions」にする必要がある（未設定）。

## 作業ルール

- 返答は日本語で行う。
- main は公開中の本番として扱う。
- main へ直接コミットや push をしない。作業ブランチで作業し、PR で main に入れる。
- 改行は LF にそろえる。`.gitattributes` の `* text=auto eol=lf` で、Git がテキストと判定したファイルをリポジトリ内でも作業ツリーでも LF にしている（各 PC の `core.autocrlf` に左右されないため）。バイナリは自動判定で変換されない。

## git フック

`.githooks/` に main を守るフックを置いている。

- `pre-commit`: 現在のブランチが main ならコミットを拒否する。detached HEAD は通す。
- `pre-push`: push 先の ref が `refs/heads/main` なら拒否する（`HEAD:main` のような指定や削除 push も含む）。

### 有効化

``` sh
git config core.hooksPath .githooks
```

clone し直したときは再度実行する。

### 注意点

- worktree では、この設定が全 worktree で共有される。ただし相対パスなので、各 worktree にチェックアウトされた `.githooks/` が使われる。そのため、フックを入れる前のコミットから切ったブランチでは効かない。
- フックをコミットするときは `git add --chmod=+x .githooks/*` で実行ビットを付ける。このリポジトリは `core.filemode=false` なので、付けないと Linux でフックが動かない。
- フックは sh で実行されるため、自動判定に任せず `.gitattributes` の `.githooks/* text eol=lf` で、必ずテキストかつ LF として扱うよう明示的に固定している（全体の `* text=auto eol=lf` より後に書いているので、こちらが優先される）。
- `--no-verify` で回避できるので、確実に止めるには GitHub のブランチ保護（ルールセット）が必要。
