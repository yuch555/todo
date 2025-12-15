# コーディング規約

## 目次

1. [ディレクトリ構成](#ディレクトリ構成)
2. [命名規則](#命名規則)
3. [TypeScript](#typescript)
4. [React/Next.js](#reactnextjs)
5. [状態管理](#状態管理)
6. [API通信](#api通信)
7. [スタイリング](#スタイリング)
8. [テスト](#テスト)
9. [コミット規則](#コミット規則)

## ディレクトリ構成

### 基本原則

- **画面・機能単位**: `app/` 配下でルーティングとページを管理
- **汎用性の判断**: 2つ以上の画面で使用される要素は汎用化を検討
- **スコープ分離**: private/public でログイン要否を明確化

### ファイル配置ルール

```
src/
├─ app/                          # Next.js App Router
│  ├─ (private)/                 # ログイン必須の画面
│  │  ├─ actions/                # 画面固有のServer Actions
│  │  ├─ apis/                   # 画面固有のAPIクライアント
│  │  ├─ components/             # 画面固有のコンポーネント
│  │  ├─ constants/              # 画面固有の定数
│  │  ├─ hooks/                  # 画面固有のカスタムフック
│  │  ├─ stores/                 # 画面固有のストア
│  │  ├─ providers/              # 画面固有のプロバイダー
│  │  ├─ schemas/                # 画面固有のバリデーションスキーマ
│  │  ├─ types/                  # 画面固有の型定義
│  │  ├─ utils/                  # 画面固有のユーティリティ
│  │  └─ layout.tsx              # private用レイアウト
│  │
│  ├─ (public)/                  # ログイン任意の画面
│  │  └─ layout.tsx              # public用レイアウト
│  │
│  └─ api/                       # Route Handlers（BFF層）
│
├─ actions/                      # 汎用Server Actions
├─ apis/                         # 汎用APIクライアント
├─ components/                   # 汎用コンポーネント
├─ constants/                    # 汎用定数
├─ hooks/                        # 汎用カスタムフック
├─ stores/                       # グローバルストア
├─ providers/                    # 汎用プロバイダー
├─ lib/                          # 外部ライブラリの初期化・設定
├─ schemas/                      # 汎用バリデーションスキーマ
├─ types/                        # 汎用型定義
└─ utils/                        # 汎用ユーティリティ関数
```

## 命名規則

### ファイル・ディレクトリ

```typescript
// ケバブケース（推奨）
user-profile.tsx
api-client.ts

// パスカルケース（コンポーネント）
UserProfile.tsx
Button.tsx

// キャメルケース（ユーティリティ）
formatDate.ts
parseJson.ts
```

### 変数・関数

```typescript
// 変数: キャメルケース
const userName = "John";
const isLoading = false;

// 定数: アッパースネークケース
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = "https://api.example.com";

// 関数: キャメルケース（動詞から始める）
function fetchUserData() {}
function validateEmail() {}
function handleSubmit() {}

// プライベート関数: アンダースコア接頭辞（必要に応じて）
function _internalHelper() {}

// 型・インターフェース: パスカルケース
type UserProfile = {};
interface ApiResponse {}

// 列挙型: パスカルケース
enum UserRole {
  Admin = "ADMIN",
  User = "USER",
}
```

### コンポーネント

```typescript
// コンポーネント: パスカルケース
export function UserProfile() {}
export const Button = () => {};

// Props: コンポーネント名 + Props
interface UserProfileProps {}
type ButtonProps = {};

// Hooks: use接頭辞
function useUserData() {}
function useAuth() {}
```

## TypeScript

### 型定義の原則

```typescript
// ✅ Good: 明示的な型定義
const userName: string = "John";
function getUserById(id: number): Promise<User> {}

// ✅ Good: 型推論を活用
const count = 0; // number型と推論される
const items = [1, 2, 3]; // number[]型と推論される

// ❌ Bad: any型の使用
const data: any = fetchData();

// ✅ Good: unknown型 + 型ガード
const data: unknown = fetchData();
if (typeof data === "object" && data !== null) {
  // 型を絞り込んで使用
}
```

### インターフェース vs Type

```typescript
// インターフェース: 拡張可能なオブジェクト型
interface User {
  id: number;
  name: string;
}

interface AdminUser extends User {
  role: "admin";
}

// Type: ユニオン、交差型、プリミティブ
type Status = "pending" | "success" | "error";
type UserWithStatus = User & { status: Status };
```

### Nullable型の扱い

```typescript
// ✅ Good: Optional Chaining & Nullish Coalescing
const userName = user?.profile?.name ?? "Guest";

// ✅ Good: 明示的なnullチェック
if (user !== null && user !== undefined) {
  console.log(user.name);
}

// ❌ Bad: 非nullアサーション演算子の乱用
const name = user!.profile!.name!;
```

### Generics

```typescript
// ✅ Good: 再利用可能な型定義
function getFirstItem<T>(items: T[]): T | undefined {
  return items[0];
}

interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}
```

## React/Next.js

### コンポーネント設計

```typescript
// ✅ Good: 関数コンポーネント（推奨）
export function UserCard({ name, email }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}

// Props定義
interface UserCardProps {
  name: string;
  email: string;
  onEdit?: () => void; // Optional
}

// ✅ Good: デフォルトProps（分割代入）
function Button({ variant = "primary", children }: ButtonProps) {
  return <button className={variant}>{children}</button>;
}
```

### Server Components vs Client Components

```typescript
// Server Component（デフォルト）
export default async function UserPage({ params }: { params: { id: string } }) {
  const user = await fetchUser(params.id);
  return <UserProfile user={user} />;
}

// Client Component（明示的に指定）
"use client";

import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### Hooks使用ルール

```typescript
// ✅ Good: カスタムフックでロジック分離
function useUserData(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then((data) => {
      setUser(data);
      setLoading(false);
    });
  }, [userId]);

  return { user, loading };
}

// ✅ Good: 依存配列の明示
useEffect(() => {
  // 処理
}, [dependency1, dependency2]);

// ❌ Bad: 依存配列の省略
useEffect(() => {
  // 処理
}); // 毎レンダリングで実行される
```

### Server Actions

```typescript
// actions/user.ts
"use server";

import { revalidatePath } from "next/cache";

export async function updateUser(formData: FormData) {
  const name = formData.get("name") as string;

  // バリデーション
  if (!name || name.length < 2) {
    return { error: "名前は2文字以上必要です" };
  }

  // データベース更新
  await db.user.update({ name });

  // キャッシュ再検証
  revalidatePath("/profile");

  return { success: true };
}
```

## 状態管理

### ストアの使い分け

```typescript
// グローバルストア: アプリ全体で共有
// stores/auth-store.ts
import { create } from "zustand";

interface AuthStore {
  user: User | null;
  setUser: (user: User | null) => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));

// 画面固有ストア: 単一画面内の複雑な状態
// app/(private)/dashboard/stores/filter-store.ts
export const useFilterStore = create<FilterStore>((set) => ({
  filters: {},
  setFilter: (key, value) =>
    set((state) => ({
      filters: { ...state.filters, [key]: value },
    })),
}));
```

### 状態管理の原則

1. **URL State**: 検索条件、ページネーション → URLパラメータ
2. **Server State**: サーバーデータ → Server Components / Server Actions
3. **Client State**: UI状態（モーダル、トグル）→ useState
4. **Global State**: 認証情報、テーマ → Zustand/Context

## API通信

### APIクライアント

```typescript
// apis/user-api.ts
import { apiClient } from "@/lib/api-client";

export async function fetchUser(id: string): Promise<User> {
  const response = await apiClient.get(`/users/${id}`);
  return response.data;
}

export async function updateUser(id: string, data: UpdateUserInput): Promise<User> {
  const response = await apiClient.put(`/users/${id}`, data);
  return response.data;
}
```

### エラーハンドリング

```typescript
// ✅ Good: 統一されたエラーハンドリング
try {
  const user = await fetchUser(id);
  return { data: user, error: null };
} catch (error) {
  if (error instanceof ApiError) {
    return { data: null, error: error.message };
  }
  return { data: null, error: "予期しないエラーが発生しました" };
}
```

### Route Handlers（BFF層）

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await fetchUser(params.id);
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json({ error: "User not found" }, { status: 404 });
  }
}
```

## スタイリング

### Tailwind CSS

```typescript
// ✅ Good: クラス名の順序（推奨）
// 1. レイアウト（flex, grid, position）
// 2. サイズ（w-, h-, p-, m-）
// 3. タイポグラフィ（text-, font-）
// 4. 色（bg-, text-, border-）
// 5. その他（rounded-, shadow-）
<div className="flex items-center justify-between w-full p-4 text-lg font-bold bg-blue-500 text-white rounded-lg shadow-md">
  Content
</div>

// ✅ Good: 条件付きクラス
import { cn } from "@/utils/cn";

<button className={cn(
  "px-4 py-2 rounded",
  variant === "primary" && "bg-blue-500 text-white",
  variant === "secondary" && "bg-gray-300 text-black"
)}>
  Click
</button>
```

### コンポーネント分離

```typescript
// ✅ Good: スタイルの共通化
// components/ui/button.tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "outline";
  size?: "sm" | "md" | "lg";
}

export function Button({ variant = "primary", size = "md", className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        "rounded font-medium transition-colors",
        {
          "bg-blue-500 text-white hover:bg-blue-600": variant === "primary",
          "bg-gray-200 text-gray-900 hover:bg-gray-300": variant === "secondary",
          "border border-gray-300 hover:bg-gray-50": variant === "outline",
        },
        {
          "px-3 py-1.5 text-sm": size === "sm",
          "px-4 py-2 text-base": size === "md",
          "px-6 py-3 text-lg": size === "lg",
        },
        className
      )}
      {...props}
    />
  );
}
```

## テスト

### テストファイル配置

```
src/
├─ components/
│  ├─ Button.tsx
│  └─ Button.test.tsx
├─ utils/
│  ├─ format.ts
│  └─ format.test.ts
```

### テスト記述

```typescript
// components/Button.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { Button } from "./Button";

describe("Button", () => {
  it("should render with text", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });

  it("should call onClick when clicked", () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByText("Click me"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## コミット規則

### Conventional Commits

```bash
# 形式
<type>(<scope>): <subject>

# 例
feat(auth): ログイン機能を実装
fix(api): ユーザー取得時のエラーを修正
docs(readme): セットアップ手順を追加
refactor(components): Button コンポーネントを分離
style(button): インデントを修正
test(auth): ログインのテストを追加
chore(deps): 依存パッケージを更新
```

### Type一覧

- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメント
- `style`: コードスタイル（動作に影響しない）
- `refactor`: リファクタリング
- `perf`: パフォーマンス改善
- `test`: テスト追加・修正
- `chore`: ビルドプロセスやツール変更

## コードレビューチェックリスト

- [ ] 型定義は適切か（`any`を使用していないか）
- [ ] コンポーネントは適切に分割されているか
- [ ] Server Component / Client Component の使い分けは適切か
- [ ] エラーハンドリングは実装されているか
- [ ] 不要なコメントやconsole.logが残っていないか
- [ ] 命名規則に従っているか
- [ ] パフォーマンスの問題はないか（不要な再レンダリングなど）
- [ ] セキュリティの問題はないか（XSS、認証不備など）
- [ ] テストは必要十分か

## 参考リンク

- [Next.js Documentation](https://nextjs.org/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Documentation](https://react.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
