# React Nativeアプリのタブナビゲーション導入ガイド

このドキュメントは、React Nativeアプリにタブナビゲーションを追加するときの設計と実装の基準です。タブの数、名前、アイコン、画面の内容はアプリの要件に合わせて決めます。

実装方法は、プロジェクトで使用しているナビゲーションライブラリに合わせて選びます。React Native自体には、すべてのプロジェクトで共通して使えるタブナビゲーションAPIはありません。

## 1. 実装前に確認すること

プロジェクトの既存構成を確認し、対応する方法を選びます。

| 既存の構成 | 実装方法 |
| --- | --- |
| Expo Routerを使用 | Expo Routerのタブレイアウトを使用する |
| React Navigationを使用 | React Navigationのタブナビゲーターを使用する |
| 別のライブラリを使用 | そのライブラリのタブAPIを使用する |
| ナビゲーションをまだ導入していない | 対象OSと必要な機能を確認してライブラリを選ぶ |

また、対象がiOS、Android、Webのどれかを確認します。同じタブ構成でも、OSによって見た目や操作が異なる場合があります。

既存の認証処理、Provider、ルート構成がある場合は、それらにタブを組み込みます。

## 2. タブの設計

タブは、アプリ内で頻繁に切り替える主要な画面への入口として設計します。具体的な画面名や数は、アプリの要件に合わせて決めてください。

各タブについて、次の情報を定義します。

| 項目 | 内容 |
| --- | --- |
| ルート名 | ナビゲーションで画面を識別する名前 |
| ラベル | 利用者に表示する名前 |
| アイコン | タブの用途を表すアイコン |
| 画面 | タブを選択したときに表示するコンポーネント |
| アクセス条件 | 認証や権限が必要か |
| 状態の扱い | タブ切り替え時に入力内容やスクロール位置を保持する必要があるか |

Expo RouterのNative Tabsでは、Androidのタブバーに最大5タブという制限があります。[Expo Native Tabs](https://docs.expo.dev/router/advanced/native-tabs/)

## 3. Expo Routerでネイティブタブを使う

この節は、**Expo Routerを設定済みのExpo SDK 58以降のプロジェクト**を対象にしています。SDK 54〜57ではインポート先やAPIに差があるため、対象SDKの公式資料に合わせてください。

Expo Routerでは、タブレイアウトと各画面をルートファイルとして作成します。

```text
app/
├── _layout.tsx
└── (tabs)/
    ├── _layout.tsx
    └── 必要なタブの画面ファイル
```

`src/app/`を使用している場合は、その配下に配置します。`(tabs)`はルートグループで、URLには含まれません。

### 3.1 ルートレイアウト

既存のルートレイアウトに、タブグループを組み込みます。以下は最小構成です。

`app/_layout.tsx`:

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen
        name="(tabs)"
        options={{ headerShown: false }}
      />
    </Stack>
  );
}
```

すでにルートレイアウトがある場合は、認証処理やProviderなどの既存構成を維持して統合してください。

### 3.2 タブレイアウト

次の例は、**1つのタブを定義するためのひな形**です。`画面のルート名`、ラベル、アイコンを実際の画面に合わせて置き換えます。

`app/(tabs)/_layout.tsx`:

```tsx
import { NativeTabs } from 'expo-router/native-tabs';

export default function TabLayout() {
  return (
    <NativeTabs>
      <NativeTabs.Trigger name="画面のルート名">
        <NativeTabs.Trigger.Label>
          タブのラベル
        </NativeTabs.Trigger.Label>

        <NativeTabs.Trigger.Icon
          sf="iOSのSF Symbols名"
          md="AndroidのMaterial Symbols名"
        />
      </NativeTabs.Trigger>

      {/* 必要なタブごとにTriggerを追加する */}
    </NativeTabs>
  );
}
```

置き換える値は次のとおりです。

- `name`：対応する画面ファイルのルート名。たとえば`index.tsx`なら`index`です。
- `Label`：タブに表示する名前
- `sf`：iOSで使用するSF Symbolsの名前
- `md`：Androidで使用するMaterial Symbolsの名前

`name`は画面ファイルのルート名と一致させます。必要な数の`NativeTabs.Trigger`と画面ファイルを用意してください。上記の文字列は説明用の値なので、置き換えてから使用します。

アイコンを含む機能の依存関係は、使用しているSDKとプロジェクト構成によって異なります。Expo Routerの導入手順と対象SDKの資料を確認してください。[Expo Routerのインストール](https://docs.expo.dev/router/installation/)

### 3.3 タブ画面

次はNative Tabsを使う場合の画面例です。各タブの画面は、対応するルートファイルにReactコンポーネントをデフォルトエクスポートします。

```tsx
import { ScrollView, Text } from 'react-native';

export default function TabScreen() {
  return (
    <ScrollView contentInsetAdjustmentBehavior="automatic">
      <Text>画面の内容をここに実装します。</Text>
    </ScrollView>
  );
}
```

画面のファイル名と、タブレイアウト内の`name`を対応させてください。検索画面を用意する場合も、ルート名を設定するだけでは検索機能や検索バーは追加されません。画面側で実装します。

### 3.4 外観とレイアウト

Native TabsはOSのネイティブUIを使用します。そのため、表示位置や外観は端末やOSによって異なります。iOS 26以降ではシステムがタブバーの背景を描画するため、背景色などの指定が反映されない場合があります。

セーフエリアの自動処理もOSごとに異なります。Androidでは下部インセットが画面に自動適用され、iOSでは最初のスクロールビューに自動インセット調整が適用されます。独自の余白処理と組み合わせる場合は、重なりや二重の余白を確認してください。[Native Tabs API](https://docs.expo.dev/versions/latest/sdk/router/native-tabs/)

Webでも同じ外観になるとは限りません。Webのレイアウトを細かく調整する場合は、Expo RouterのWeb向けタブ構成も検討してください。

## 4. Expo Router以外で実装する

React Navigationなどの別ライブラリでは、タブの登録方法、画面の接続方法、依存関係が異なります。上記の`NativeTabs`コードをそのまま使うことはできません。

React Navigationを使用している場合は、プロジェクトのバージョンに対応した公式資料に従って、タブナビゲーターへ画面を登録します。既存の`NavigationContainer`やProviderがある場合は、現在のナビゲーション構成に統合してください。

- [React Navigation: Bottom Tabs](https://reactnavigation.org/docs/bottom-tab-navigator/)
- [React Navigation: Native Bottom Tabs](https://reactnavigation.org/docs/native-bottom-tab-navigator/)

Native Bottom Tabsを選ぶ場合は、対象OS、React Native、Expo SDK、ネイティブ依存関係の条件を確認します。通常のBottom TabsとはAPIが異なります。

## 5. 認証とデータ表示

タブを表示することと、その画面へのアクセスを許可することは別に設計します。認証や権限が必要な場合は、画面への遷移とデータ取得の両方で適切に扱います。

ユーザーごとのデータを表示する場合は、少なくとも以下の状態を用意します。

- 読み込み中
- データなし
- 未認証
- 取得エラー
- 表示可能

権限や残高などの重要な判定はサーバー側でも行います。クライアントから送られた利用者IDや金額だけで確定しないでください。

## 6. 導入後の確認

対象とするOSと端末で、次の項目を確認します。

- 各タブを選択すると、対応する画面が開く
- ラベル、アイコン、選択状態が認識できる
- 画面とタブバー、システム領域が重なっていない
- キーボード表示時に入力欄を操作でき、タブバーの表示位置が要件に合っている
- 文字サイズを変更しても、タブを読んで操作できる
- 戻る操作やタブの再選択時の挙動が要件に合っている
- タブ切り替え時の画面状態が要件どおりに保持または初期化される
- 認証状態やデータ取得エラーに応じた画面が表示される

Expoプロジェクトでは、依存関係の整合性を`npx expo install --check`や`npx expo-doctor`で確認できます。Expo Goで使えるネイティブ機能はSDKとアプリに含まれる依存関係に左右されるため、必要に応じて開発ビルドを使用してください。
