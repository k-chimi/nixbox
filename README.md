# nixbox

[Nix](https://nix.dev/) 環境をお試しするためのコンテナを生成するツール。

- Nix 環境の一時的な作成
- 効率的なキャッシュの使いまわし
- 申し訳程度のシームレスなお試し体験

Podman または Docker そして Bash が動く環境であることが前提です。

> [!NOTE]
> ホスト環境が Linux である場合、[Toolbx](https://containertoolbx.org/) に Nix をインストールするほうがより学習に適しているかもしれません。

<details>
<summary>

**使い方**: `$ ./nixbox -h`
</summary>

```
Usage: nixbox [-h]
       nixbox <up|down|enter-daemon|enter>

  Create Nix daemon container for otameshi-ing Nix.

subcommands:

  up ... Create and start Nix daemon container.
  down ... Stop and destroy Nix daemon container.
  start ... Start Nix daemon container.
  stop ... Stop Nix daemon container.
  enter-daemon ... Enter running Nix daemon container for debug.
  enter ... Enter temporary Nix container.

Environments:

  NIXBOX_NAME="nixbox-daemon-container" ... container name.
  NIXBOX_IMAGE="nixos/nix" ... container image for Nix daemon.
  NIXBOX_TEMPORARY_IMAGE="busybox" ... container image for temporary environment.

Volume names:

  "nixbox-daemon-container_etc-nix" ... (container) /etc/nix
  "nixbox-daemon-container_nix" ... (container) /nix
```
</details>

このツールは、`nix-daemon` を動かす単一のコンテナと、そのコンテナに対してビルド等を要求する開発環境を提供します。

環境変数を用いてコンテナ名などをカスタマイズすることができるので、うまく動かせば複数のデーモンを立ち上げられるかもしれません。

## とりあえず動かしてみるコーナー

### `Hello, world!`

```sh
# `nix-daemon` を立ち上げる
./nixbox up

# 一時的な開発環境コンテナを生成し、`nix-shell -p hello --run hello` を実行する
./nixbox enter -c nix-shell -p hello --run hello

# Output:
# Hello, world!
```

### PHP 8.5

```sh
# php85 が入ったシェル環境に切り替わる。
# このツールでは現在のディレクトリを bind mount するので、そのままコードを実行したりなどの作業ができる。
./nixbox enter -c nix-shell -p php85
```

### nix-command

デフォルトで nix-command が有効化されているので、従来通りのハイフン区切りのコマンドから、空白区切りのコマンド形式を扱うこともできます。

```sh
./nixbox enter -c nix shell "nixpkgs#nodejs_22"
```

## Nix ってなんなのさ！

Nix とは再現性の高いビルドシステムとして開発されているパッケージマネージャーのようなもの。

何が嬉しいかと言えば、パッケージのビルド方法として環境の整え方を残しておけるので、一度書いてしまえば面倒な作業が必要なくなる事です。

また、[nixpkgs](https://search.nixos.org/packages) という Nix 用のパッケージレジストリでは PHP や NodeJS、Ripgrep や Sudo など。<br/>
nixpkgs にはいろんなものが揃っているので、何かを試したい場合は検索窓を叩いてみると良いキッカケが生まれるかもしれません。

Nix をパッケージマネージャーとして利用した [Nix OS <sup>(Wikipedia)</sup>](https://ja.wikipedia.org/wiki/NixOS) という Linux ディストリビューションもあるので、Nix が実現しようとしている再現性というものの本気度が伺い知れるかもしれません。

<details>
<summary>

### 個人的に Nix を触る上でつまずいているところ
</summary>

- パッケージや環境の設定の書き方について

  Nix は DSL としてパッケージを表すスクリプト言語を持っていますが、[flakes](https://nix.dev/manual/nix/2.34/command-ref/new-cli/nix3-flake) やパッケージ用の形式など、書く内容がそれぞれバラバラの形式があるためドキュメントを読む際などは注意が必要かもしれません。

  AI に『この書き方は Nix では、どの定義ファイルの種類で書くべきもの？』のような質問をすると、聞き馴染みのないツールの割には正確に答えてくれました。

- DSL の形式について

  Nix が持つ言語は Nix 言語と呼ばれていることを良く目にします。　この Nix 言語は Lua などの手続きパラダイムの言語と比べると大変異質であり、初見では何がなんだか分かりづらいかもしれません。

  それもそのはず、 Nix 言語は関数型のパラダイムを持っている言語である上、関数定義の書き方を特殊なので、このドキュメントを読むと混乱が解消できると思います。

  https://nix.dev/manual/nix/2.34/language/index.html

- パッケージの実体について

  Nix は Nix Store というバイナリ置き場を持っています。　これは不変であり、バイナリ置き場そのものは紐付けられたハッシュと矛盾がないように Nix が管理しています。

  Nix で言語のランタイムを管理している場合に NPM のグローバルインストールや、PIP によるパッケージ管理を行うと、コマンドの実行に失敗するなどの不具合が現れるかも知れません。

  その場合、そのパッケージマネージャーでインストールするパッケージも Nix の管理下に配置できるようにパッケージや [flakes](https://nix.dev/manual/nix/2.34/command-ref/new-cli/nix3-flake) を用いたり、<br/>
  Nix Store の実体である `/nix/store` にパッケージマネージャーが書き込もうとしないように設定を変更して回避するなどの方法があります。
</details>
