# 今日学んだこと: 固有情報をコミットしちゃった時のGit履歴修正

*2025-07-12*

## 🎯 事件の概要

素晴らしいプルリクを受け取って取り込んだ後、固有情報（実際のドメイン名、IPアドレス、ユーザー名）がファイルに残っていることに気づいた事件。

## 📋 発生した問題

### 固有情報の残存
- **bon-soleil.com**: 8ファイルに残存
- **54.65.29.88**: 実際のIPアドレス
- **ec2-user**: 実際のユーザー名
- **パス情報**: `/home/ec2-user/` 等

### Git履歴の複雑化
```
bfd7510 今日の完全記録（最後のクリーンなコミット）
├─ b5eb9e2 Apache vhost管理ツール開発...
├─ 6417d16 vhost管理ツール実装完了... ← 固有情報あり
├─ 717ce70 友人向けGit運用教育ファイル設置
├─ 666895b PLEASE_READ.mdにドメイン管理... ← 固有情報あり
└─ 1d404c2 Apache vhost管理ツールのテンプレート化... ← 修正版
```

## 💡 学んだ対処法

### 1. 固有情報の一括置換
```bash
# macOSのsed形式で一括置換
find . -name "*.md" -type f -exec sed -i '' 's/bon-soleil\.com/example.com/g' {} \;
find . -name "*.md" -type f -exec sed -i '' 's/54\.65\.29\.88/{EIP}/g' {} \;
find . -name "*.md" -type f -exec sed -i '' 's/\/home\/ec2-user\//\/home\/{username}\//g' {} \;
```

### 2. Git履歴の整理方法

#### 試行錯誤したアプローチ
- **インタラクティブリベース**: 各コミットの手動修正（面倒）
- **filter-branch**: 一括処理（複雑）
- **squash**: 全部まとめて1コミットに（採用✅）

#### 最終的な解決方法
```bash
# bfd7510まで戻って全部squash
git reset --soft bfd7510
git commit -m "Apache vhost管理ツール開発完了とideanotes実用性完全実証"
```

### 3. 検証方法
```bash
# 修正前後で内容が同じか確認
git diff 7f0b43a HEAD  # 差分なし = 成功
```

## 🛡️ 予防策

### コミット前チェックリスト
- [ ] 実際のドメイン名が含まれていないか
- [ ] IPアドレスが露出していないか  
- [ ] ユーザー名やパス情報が含まれていないか
- [ ] その他の機密情報がないか

### 設計時の配慮
- **最初からテンプレート化**: `example.com`, `{username}` 等を使用
- **.gitignore活用**: 機密設定ファイルの除外
- **設定ファイル分離**: template/local パターンの採用

### レビュープロセス
- **プルリク時の確認**: 受け取る側もチェック
- **自動チェック**: CI/CDでの機密情報検出
- **定期監査**: 既存ファイルの定期確認

## 🔄 Git運用の学び

### reflogとcommit historyの違い
- **commit history**: パブリックに見える正式な履歴
- **reflog**: ローカルの作業履歴（自動削除される）

### force-pushの適切な使用
- **--force-with-lease**: 安全な強制更新
- **履歴書き換え後**: 必須の操作

### squashの効果
- **複雑な履歴を整理**: 複数コミットを1つに統合
- **機密情報の完全除去**: 中間状態を履歴から削除
- **意味のある単位**: 論理的なまとまりでコミット

## 📊 結果

### 技術的成果
- ✅ **固有情報完全削除**: 8ファイル、26箇所修正
- ✅ **Git履歴整理**: squashで美しい1コミットに統合
- ✅ **内容検証**: 修正前後で差分ゼロを確認

### プロセス改善
- ✅ **チェック体制**: コミット前の確認習慣
- ✅ **テンプレート化**: 最初から汎用化を意識
- ✅ **協力体制**: 相互レビューの重要性認識

## 🎯 今後への活かし方

### 開発フロー改善
1. **設計段階**: 固有情報を使わない前提で設計
2. **実装段階**: template/local分離パターン採用
3. **コミット段階**: 機密情報チェックの習慣化
4. **レビュー段階**: 受け取る側も責任を持つ

### ツール活用
- **pre-commit hook**: 機密情報の自動検出
- **git-secrets**: AWS等の機密情報検出
- **定期監査スクリプト**: 既存ファイルのチェック

## 🤔 メタ的考察

### ideanotesらしさ
- **失敗も学習機会**: 問題発生→対処→記録→改善
- **予想外も歓迎**: 想定外のトラブルから貴重な学び
- **実践的知識**: 実際に遭遇した問題の対処法

### 協力関係の価値
- **友人のプルリク**: 高品質だったが機密情報チェックは不十分
- **相互補完**: 技術力＋セキュリティ意識の組み合わせ
- **建設的解決**: 責任追及より改善策に集中

## 📝 友人への伝達事項（直接連絡予定）

「プルリクは完璧だったけど、固有情報のチェックも今度からお願いします😅  
今回は良い学習になったので、お互いに気をつけましょう！」

---

*結論: 固有情報をコミットしちゃうと後が大変。最初からテンプレート化が正解*