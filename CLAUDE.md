# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

自分の Web ページ（GitHub Pages ユーザーサイト `biuntri.github.io`）のリポジトリ。

## 現状

- ソース、ビルド、テストはまだない。
- 技術スタックは未定。決まったら、このファイルにコマンドと構成を書き足すこと。
- 推測でコマンドや構成を書かない。

## 公開方法

未定（技術スタックと合わせて決める）。

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
