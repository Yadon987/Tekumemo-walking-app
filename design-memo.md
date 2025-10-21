# 💻 1. モデル設計（てくメモ） v1.0

## 📊 プロジェクト基本情報
- **作成日**: 2025/10/21
- **最終更新**: 2025/10/21
- **ステータス**: モデル設計開始

## モデル構成図（イメージ）
アプリの主要なエンティティとして、以下の3つのモデルを定義します。

| モデル名 | 役割 |
| :--- | :--- |
| **User (ユーザー)** | ログインユーザー、投稿者、ランキング参加者を管理 |
| **Post (投稿)** | ユーザーが散歩中に思いついたアイデアやコメントを管理 |
| **WalkLog (歩行記録)** | ユーザーの毎日の歩数、距離、時間などの活動データを管理 |

---

## 1. User モデル (ユーザー)

| カラム名 | データ型 | 説明 | 備考 |
| :--- | :--- | :--- | :--- |
| `id` | `integer` | プライマリキー | |
| `name` | `string` | ユーザー名 | |
| `email` | `string` | ログイン用メールアドレス | |
| `password_digest` | `string` | パスワードのハッシュ値 | |
| `google_uid` | `string` | Googleログイン時のID | **【高優先学習要素】** |
| `profile` | `text` | プロフィール文 | |
| `created_at` | `datetime` | 作成日時 | |
| `updated_at` | `datetime` | 更新日時 | |

**関連付け:**
* `has_many :posts` (投稿)
* `has_many :likes` (いいね)
* `has_many :walk_logs` (歩行記録)
* `has_many :favorites` (お気に入り/後で読む)

---

## 2. Post モデル (投稿：アイデア/コメント)

| カラム名 | データ型 | 説明 | 備考 |
| :--- | :--- | :--- | :--- |
| `id` | `integer` | プライマリキー | |
| `user_id` | `integer` | 投稿したユーザーID | Userとの関連付け |
| `content` | `text` | アイデアや一言コメントの内容 | |
| `is_public` | `boolean` | 公開/非公開設定 | |
| `created_at` | `datetime` | 作成日時 | |
| `updated_at` | `datetime` | 更新日時 | |

**関連付け:**
* `belongs_to :user`
* `has_many :likes`
* `has_many :favorites`

> *追加モデル: PostとWalkLogを紐づけることも可能ですが、まずはシンプルにPost単体で設計します。*

---

## 3. WalkLog モデル (歩行記録・ランキング用データ)

このモデルは、ランキング機能と歩数・距離計算のデータ基盤となります。

| カラム名 | データ型 | 説明 | 備考 |
| :--- | :--- | :--- | :--- |
| `id` | `integer` | プライマリキー | |
| `user_id` | `integer` | ユーザーID | Userとの関連付け |
| `date` | `date` | 記録日 | 日ごとの集計に利用 |
| `steps` | `integer` | 歩数 | ランキングの基準 |
| `distance_km` | `float` | 距離（km） | |
| `duration_min` | `integer` | 歩行時間（分） | |
| `weather` | `string` | 記録日の天気（外部API連携） | **【MVP構築要素】** |
| `created_at` | `datetime` | 作成日時 | |
| `updated_at` | `datetime` | 更新日時 | |

**関連付け:**
* `belongs_to :user`

---

## 💡 4. コア機能を実現する追加モデル

「いいね」とお気に入り機能は、上記3つのモデル間の**多対多**の関係を仲介するために、中間テーブルとして定義します。

### 4.1. Like モデル (いいね機能)

| カラム名 | データ型 | 説明 | 備考 |
| :--- | :--- | :--- | :--- |
| `id` | `integer` | プライマリキー | |
| `user_id` | `integer` | いいねを押したユーザー | |
| `post_id` | `integer` | いいねされた投稿 | |

**関連付け:**
* `belongs_to :user`
* `belongs_to :post`

### 4.2. Favorite モデル (お気に入り機能)

| カラム名 | データ型 | 説明 | 備考 |
| :--- | :--- | :--- | :--- |
| `id` | `integer` | プライマリキー | |
| `user_id` | `integer` | お気に入りしたユーザー | |
| `post_id` | `integer` | お気に入りされた投稿 | |

**関連付け:**
* `belongs_to :user`
* `belongs_to :post`

## ✅ 進捗管理
- [x] コアエンティティ定義
- [x] カラム定義（ドラフト）
- [x] 関連付け定義（has_many/belongs_to）
- [ ] 補助テーブル定義
- [ ] カラム精査と制約確認
- [ ] 画面遷移図の作成
- [ ] ER図の作成

## 📝 変更履歴
- v1.0: モデル設計開始
