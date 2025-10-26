# 💻 1. モデル・テーブル設計（てくメモ） v1.2

## 📊 プロジェクト基本情報
- **作成日**: 2025/10/21
- **最終更新**: 2025/10/23
- **ステータス**: ER図の作成

てくメモアプリでは、ユーザーの散歩習慣の継続をサポートするため、投稿・記録・交流の3つの観点からデータモデルを設計しました。

---

### テーブル一覧

#### 1. **Users（ユーザー）**
ユーザーの基本情報を管理するテーブル。Googleログイン認証に対応。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| user_id | INTEGER | PK, AUTO_INCREMENT | ユーザーID |
| google_id | VARCHAR | UNIQUE | Googleログイン用ID |
| username | VARCHAR | NOT NULL | ユーザー名 |
| email | VARCHAR | UNIQUE, NOT NULL | メールアドレス |
| profile_image | VARCHAR | - | プロフィール画像URL |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |
| updated_at | TIMESTAMP | DEFAULT NOW() | 更新日時 |

---

#### 2. **Posts（投稿）**
散歩中の気づきやアイデアを共有するための投稿テーブル。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| post_id | INTEGER | PK, AUTO_INCREMENT | 投稿ID |
| user_id | INTEGER | FK → Users, NOT NULL | 投稿者ID |
| title | VARCHAR | NOT NULL | 投稿タイトル |
| content | TEXT | - | 投稿内容（気づき・アイデア） |
| steps_count | INTEGER | - | その日の歩数 |
| distance_km | DECIMAL(5,2) | - | 歩行距離（km） |
| walk_date | DATE | NOT NULL | 散歩実施日 |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |
| updated_at | TIMESTAMP | DEFAULT NOW() | 更新日時 |

**INDEX**: user_id, walk_date, created_at

---

#### 3. **Likes（いいね）**
投稿に対するいいね機能を実現するテーブル。ユーザーのエンゲージメント向上を目的とする。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| like_id | INTEGER | PK, AUTO_INCREMENT | いいねID |
| user_id | INTEGER | FK → Users, NOT NULL | いいねしたユーザーID |
| post_id | INTEGER | FK → Posts, NOT NULL | いいね対象の投稿ID |
| created_at | TIMESTAMP | DEFAULT NOW() | いいね日時 |

**UNIQUE制約**: (user_id, post_id) - 重複いいね防止
**INDEX**: post_id

---

#### 4. **Favorites（お気に入り）**
気に入った投稿を保存し、後から見返せるようにするテーブル。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| favorite_id | INTEGER | PK, AUTO_INCREMENT | お気に入りID |
| user_id | INTEGER | FK → Users, NOT NULL | お気に入り登録したユーザーID |
| post_id | INTEGER | FK → Posts, NOT NULL | お気に入り対象の投稿ID |
| created_at | TIMESTAMP | DEFAULT NOW() | 登録日時 |

**UNIQUE制約**: (user_id, post_id) - 重複登録防止
**INDEX**: post_id

---

#### 5. **WalkRecords（散歩記録）**
個人の散歩データを蓄積し、ランキング機能やモチベーション維持に活用するテーブル。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| record_id | INTEGER | PK, AUTO_INCREMENT | 記録ID |
| user_id | INTEGER | FK → Users, NOT NULL | ユーザーID |
| steps_count | INTEGER | NOT NULL | 歩数 |
| distance_km | DECIMAL(5,2) | - | 歩行距離（km） |
| walk_date | DATE | NOT NULL | 散歩実施日 |
| duration_minutes | INTEGER | - | 散歩時間（分） |
| created_at | TIMESTAMP | DEFAULT NOW() | 記録日時 |

**INDEX**: user_id, walk_date, (user_id, walk_date)

---

#### 6. **Notifications（通知）**
いいねやお気に入り登録時にユーザーへ通知を送るためのテーブル。

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|------|------|
| notification_id | INTEGER | PK, AUTO_INCREMENT | 通知ID |
| user_id | INTEGER | FK → Users, NOT NULL | 通知先ユーザーID |
| message | TEXT | NOT NULL | 通知メッセージ |
| is_read | BOOLEAN | DEFAULT FALSE | 既読フラグ |
| notification_type | VARCHAR | - | 通知種別（like, favorite等） |
| created_at | TIMESTAMP | DEFAULT NOW() | 通知日時 |

**INDEX**: user_id, is_read, created_at

---

### 🔗 リレーションシップ（ER図）
```
Users (1) ─────< (N) Posts
  │                    │
  │                    ├───< (N) Likes
  │                    └───< (N) Favorites
  │
  ├───< (N) Likes
  ├───< (N) Favorites
  ├───< (N) WalkRecords
  └───< (N) Notifications
```

**主要なリレーション**:
- Users → Posts (1:N): 1ユーザーは複数の投稿を作成できる
- Users → Likes (1:N): 1ユーザーは複数の投稿にいいねできる
- Posts → Likes (1:N): 1投稿は複数のいいねを受け取れる
- Users → Favorites (1:N): 1ユーザーは複数の投稿をお気に入り登録できる
- Posts → Favorites (1:N): 1投稿は複数のユーザーにお気に入り登録される
- Users → WalkRecords (1:N): 1ユーザーは複数の散歩記録を持つ
- Users → Notifications (1:N): 1ユーザーは複数の通知を受け取る

---

### 💡 設計のポイント

#### データの正規化と分離
- **PostsとWalkRecordsの分離**: 投稿は「他者と共有したい記録」、WalkRecordsは「個人の継続データ」として明確に分離しました。これにより、ランキング機能では純粋な歩数データのみを集計でき、投稿機能とは独立して運用可能です。

#### パフォーマンスとスケーラビリティ
- **適切なインデックス設計**: 頻繁に検索されるカラム（user_id、walk_date、created_at等）にはインデックスを設定し、クエリパフォーマンスを最適化しています。
- **UNIQUE制約の活用**: LikesとFavoritesには(user_id, post_id)の複合ユニーク制約を設定し、重複データの防止とデータ整合性を保証しています。

#### ユーザー体験の向上
- **通知機能の実装**: Notificationsテーブルにより、いいねやお気に入り登録時にリアルタイムで通知を送信し、ユーザーエンゲージメントを高めます。
- **既読管理**: is_readフラグにより、未読通知の管理が可能です。

#### 拡張性の確保
- **外部キー制約**: すべてのリレーションに外部キー制約を設定し、参照整合性を保証。データの信頼性を担保しています。
- **タイムスタンプの記録**: created_atとupdated_atを全テーブルに実装し、データの追跡と監査が可能です。

---

### 🎯 想定される主要クエリ

#### ランキング取得
```sql
SELECT user_id, SUM(steps_count) as total_steps
FROM walk_records
WHERE walk_date BETWEEN '2025-10-01' AND '2025-10-31'
GROUP BY user_id
ORDER BY total_steps DESC
LIMIT 10;
```

#### 人気投稿の取得
```sql
SELECT p.*, COUNT(l.like_id) as like_count
FROM posts p
LEFT JOIN likes l ON p.post_id = l.post_id
GROUP BY p.post_id
ORDER BY like_count DESC
LIMIT 20;
```

#### ユーザーのお気に入り一覧
```sql
SELECT p.*
FROM posts p
INNER JOIN favorites f ON p.post_id = f.post_id
WHERE f.user_id = ?
ORDER BY f.created_at DESC;
```

## ✅ 進捗管理
- [x] コアエンティティ定義
- [x] カラム定義（ドラフト）
- [x] 関連付け定義（has_many/belongs_to）
- [x] 補助テーブル定義
- [x] カラム精査と制約確認
- [x] PC画面遷移図の作成
- [ ] スマホ画面遷移図の作成
- [x] 不足や誤差の修正
- [x] ER図の作成

## 📝 変更履歴
- v1.0: モデル・テーブル設計開始
- v1.1: PC画面遷移図の作成
- v1.2: ER図の作成
- v1.3: ER図の修正←いいね機能をリアクション機能に変更/
