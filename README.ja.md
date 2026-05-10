# sigrok-mcp

ロジックアナライザ・オシロスコープを AI から自然言語で操作するための MCP サーバです。
[libsigrok](https://sigrok.org/) の上に薄いレイヤーを乗せることで、200 機種以上の測定器を Claude Desktop などの MCP クライアントから扱えるようにします。

## これは何か

測定器の世界には「ハードウェアと専用ソフトが 1 対 1 で縛られている」という長年の課題があります。安価なロジアナや USB オシロを買っても、付属ソフトが古かったり、特定の OS でしか動かなかったり、機能が限られていたりします。

この問題に対する既存の答えが [sigrok](https://sigrok.org/) プロジェクトです。sigrok は 200 以上の機種を共通の API で扱えるようにする OSS で、PulseView という GUI も提供しています。

sigrok-mcp はそこに「AI から呼び出せる入口」を加えるプロジェクトです。

```
あなた  ──→  Claude Desktop  ──→  sigrok-mcp  ──→  libsigrok  ──→  実機
```

自然言語で測定の意図を伝えると、Claude が適切な道具を選び、libsigrok 経由で実機を操作し、結果を解釈して返してくれます。

## 何ができるか

最終的に提供する道具のイメージです。

- `list_devices`: 接続されている測定器を一覧表示
- `capture_waveform`: 指定設定で波形を取得
- `get_waveform_summary`: 波形の特徴量（周波数、振幅など）を返す
- `decode_protocol`: I2C / SPI / UART などのプロトコル解読
- `export_waveform`: 波形ファイルとして保存

これらを組み合わせると、たとえばこういう会話が成立します。

> あなた: センサーから出てる I2C の信号、何か変な気がする。見てくれる？
>
> Claude: 1 秒間、Ch0 をキャプチャしました。I2C として解読すると、0x68 → 0x0B → 0xFF という通信が見えますが、途中で ACK が来ていない箇所が 3 回あります。スレーブが応答できていないか、配線が緩んでいる可能性があります。

## なぜ MCP サーバなのか

GUI を自分で作ると開発量が膨大になりますが、MCP サーバなら Claude Desktop などの既存クライアントをそのまま UI として使えます。開発のスコープを「ハードウェアの抽象化レイヤー」だけに集中できるのが大きな利点です。

加えて、AI と測定器の連携はまだ世界でも数えるほどしかなく、今ここに OSS を置く価値があります。

## 設計上の原則

LLM の確率的な性質と、測定の世界が要求する決定論性は本来相性が悪い領域です。この衝突を避けるため、sigrok-mcp では以下を原則とします。

- **LLM は意図の翻訳と段取りに専念する**: 「波形を取って」を「どの道具をどの順で呼ぶか」に変換する仕事だけを LLM がやる
- **ハードウェアへの命令は決定論的なコードで実行する**: MCP ツールの中身は普通のコードであり、LLM は介在しない
- **危険な操作には人間の承認を挟む**: 後述の安全機構で、想定外の挙動を防ぐ

## 対応機種

libsigrok がサポートする全機種が対象です。最初の動作確認機種は以下を想定しています。

- 開発初期: libsigrok 付属の **demo driver**（仮想デバイス、実機不要）
- 最初の実機: **Hantek 6022BE**（数千円の USB オシロ、sigrok 対応で実績あり）

## インストール

```bash
pip install sigrok-mcp
```

Claude Desktop の MCP 設定に追加:

```json
{
  "mcpServers": {
    "sigrok": {
      "command": "sigrok-mcp"
    }
  }
}
```

詳細は [INSTALL.md](./INSTALL.md) を参照してください。

## ライセンス

GPLv3。libsigrok のライセンスを継承しています。

## 関連プロジェクト

- [sigrok](https://sigrok.org/) - 本プロジェクトが依存する測定器抽象化レイヤー
- [PulseView](https://sigrok.org/wiki/PulseView) - sigrok 公式の GUI フロントエンド
- [Model Context Protocol](https://modelcontextprotocol.io/) - Anthropic が提唱する AI ツール接続プロトコル

## 開発状況

現在: **設計フェーズ**。実装はこれから。
進捗や設計議論は [DESIGN.md](./DESIGN.md) を参照してください。

## コントリビュート

歓迎します。特に、自分の手元の機種で動作確認してくださる方を募集しています。詳細は [CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください（準備中）。
