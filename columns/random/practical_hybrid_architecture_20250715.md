# 実践的ハイブリッドアーキテクチャ論 - なぜ私はEC2を手放さないのか

*2025-07-15*

## 🎯 サーバーレス全盛時代の異端児

「え、まだEC2使ってるの？」

そう言われることもある。でも、私は確信を持って答える。「はい、使ってます」と。

## 🏗️ 実際の構成

私のAlexa Voice Memoプロジェクトの構成：

```
[ユーザー] → [EC2/Apache] → [Lambda] → [DynamoDB]
                ↓
            [certbot/SSL]
```

**EC2の役割**：
- SSL証明書の管理（certbot）
- リバースプロキシ
- 開発環境のホスティング

**なぜこの構成？**

## 💡 開発者視点の現実

### 1. 慣れた技術の生産性

```bash
# 新しいサブドメインを追加
sudo vim /etc/apache2/sites-available/new-feature.conf
# 5分で完了

# vs

# API Gateway + CloudFrontの設定
# AWS Console → 複数画面 → JSON設定 → デプロイ → 祈る
# 1時間経過...まだ動かない
```

### 2. デバッグのしやすさ

```bash
# EC2なら
ssh my-server
tail -f /var/log/apache2/error.log
# 問題が即座に見える

# フルサーバーレスなら
# CloudWatch → ログストリーム探し → 時系列で追跡 → 
# 「あれ、このログどこ行った？」
```

### 3. 柔軟性の価値

vhost一つで何でもできる：
- `dev.example.com` → 開発中の新機能
- `staging.example.com` → クライアント確認用
- `demo.example.com` → デモ環境
- `api-v2.example.com` → API新バージョンテスト

全部SSL付き、全部5分で設定完了。

## 🤔 「それってアンチパターンじゃない？」

**よく言われる批判：**
- 「スケーラビリティは？」→ まだ必要ない
- 「可用性は？」→ 個人プロジェクトには過剰
- 「コストは？」→ t3.microで月$10、全然OK
- 「モダンじゃない」→ 動けばいい

## 🎓 学んだこと

### 技術選定の基準

1. **習熟度 > 新しさ**
   - 使い慣れた技術の方が結果的に速い
   
2. **シンプルさ > 完璧さ**
   - 複雑な構成は後からでも移行できる
   
3. **実用性 > 理想論**
   - まず動くものを作ることが大事

### EC2の隠れた価値

**開発基盤として**：
- Claude Codeの実行環境に最適
- 手元の端末はSSHクライアントだけでOK
- どこからでも同じ環境で開発可能

**運用の簡便性**：
- 何か起きてもSSHで入って直接対処可能
- ログが全部一箇所に集まる
- 設定ファイルをGit管理できる

## 🚀 ハイブリッドの美学

**いいとこ取り戦略**：
- コアロジック → Lambda（スケーラブル）
- データ → DynamoDB（マネージド）
- フロント → EC2（コントロール可能）

これは妥協ではなく、**意図的な選択**。

## 📊 実績が証明

- Alexa Voice Memo：問題なく稼働中
- 開発速度：アイデアから公開まで最速
- 運用負荷：ほぼゼロ
- 満足度：100%

## 💭 結論

**「ベストプラクティス」は状況による**

スタートアップの初期、個人プロジェクト、プロトタイプ開発...
こういう場面では、EC2 + 慣れた技術スタックが最適解になることも多い。

大事なのは：
1. 自分の状況を正しく認識する
2. 目的に合った技術を選ぶ
3. 必要になったら移行する勇気を持つ

**完璧なサーバーレスより、動く不完全。**

これが私の開発哲学。

---

*技術の理想と現場の現実、そのギャップを埋めるのが真のエンジニアリング*