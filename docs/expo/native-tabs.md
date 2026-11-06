# Expo アプリの下部タブナビゲーション

Expo Router の Native Tabs を使う実装例です。Expo SDK 58 と、Expo Router の設定が完了したプロジェクトを前提にしています。

## 依存関係の確認

```sh
npm install
npx expo install --check
npx expo-doctor
```

## ファイル構成

```text
app/
├── _layout.tsx
└── (tabs)/
    ├── _layout.tsx
    ├── index.tsx
    ├── explore.tsx
    ├── search.tsx
    └── profile.tsx
```

`src/app/` を使用している場合は、その配下に配置します。`(tabs)` は URL に含まれないルートグループです。

## ルートレイアウト

`app/_layout.tsx`:

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    </Stack>
  );
}
```

既存の認証処理や Provider がある場合は、それらを維持して組み込みます。

## タブレイアウト

`app/(tabs)/_layout.tsx`:

```tsx
import { NativeTabs } from 'expo-router/native-tabs';

export default function TabLayout() {
  return (
    <NativeTabs
      tintColor="#222222"
      iconColor={{ default: '#777777', selected: '#222222' }}
      backgroundColor="#FFFFFF"
      indicatorColor="#EEEEEE"
    >
      <NativeTabs.Trigger name="index">
        <NativeTabs.Trigger.Label>ホーム</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf={{ default: 'house', selected: 'house.fill' }} md="home" />
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="explore">
        <NativeTabs.Trigger.Label>見つける</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf={{ default: 'safari', selected: 'safari.fill' }} md="explore" />
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="search">
        <NativeTabs.Trigger.Label>検索</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="magnifyingglass" md="search" />
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="profile">
        <NativeTabs.Trigger.Label>プロフィール</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon
          sf={{ default: 'person.crop.circle', selected: 'person.crop.circle.fill' }}
          md="account_circle"
        />
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```

`name` は対応するルート名に合わせます。`sf` は iOS の SF Symbols、`md` は Android の Material Symbols です。この例では Android の選択状態は色で表します。

`indicatorColor` は Android に適用されます。iOS 26 以降はシステムが背景を描画するため、`backgroundColor` は反映されません。Liquid Glass などの外観は OS とビルド環境に左右されます。

`name="search"` だけでは検索機能や検索バーは追加されません。別途実装してください。

SDK 54〜57 ではインポート先が `expo-router/unstable-native-tabs` です。SDK によって API も異なるため、対応する公式資料を確認してください。

## 画面の例

`app/(tabs)/explore.tsx`:

```tsx
import { ScrollView, Text } from 'react-native';

export default function ExploreScreen() {
  return (
    <ScrollView contentInsetAdjustmentBehavior="automatic">
      <Text>コンテンツをここに表示</Text>
    </ScrollView>
  );
}
```

ほかの画面にもコンポーネントを作成し、デフォルトエクスポートします。

Android では Native Tabs が下部インセットを自動適用しますが、上部などは画面側で対応する必要があります。iOS では最初のスクロールビューに自動インセット調整が適用されます。ヘッダーの有無に合わせて調整し、余白の不足や二重適用を確認してください。

## 利用者ごとのデータ

1. データの管理元と利用者の認証方法を決めます。
2. API の応答型と、読み込み中・データなし・未認証・エラー状態を定義します。
3. 認証済み利用者に紐づく情報をサーバーから取得します。
4. サーバー側で権限を確認します。クライアントから送られた利用者 ID や金額だけで、権限や残高、特典を確定しません。
5. 仮の個人情報や金額を本物のように表示しません。デモデータにはその旨を明記します。

## 起動と確認

Expo Go:

```sh
npx expo start --go
```

開発ビルドは、対応するビルドを端末にインストールしてから起動します。

```sh
npx expo start --dev-client
```

Expo Go では iPhone のカメラ、Android の Expo Go 内の QR スキャナーで開きます。実機の iPhone では CLI と Expo Go を同じ Expo アカウントでログインしてください。通常はパソコンと端末を同じ Wi-Fi に接続します。

Expo Go は対応 SDK に制約があります。SDK が対応しない場合や、Expo Go に含まれないネイティブ機能を追加する場合は開発ビルドを使用します。

両 OS でタブの移動、ラベル、アイコン、選択状態、セーフエリアを確認します。キーボード表示時や文字サイズ変更時の操作も確認してください。

## 参考

- [Native Tabs](https://docs.expo.dev/router/advanced/native-tabs/)
- [Native Tabs API](https://docs.expo.dev/versions/latest/sdk/router/native-tabs/)
- [Expo Router のインストール](https://docs.expo.dev/router/installation/)
- [ルーティング記法](https://docs.expo.dev/router/basics/notation/)
- [Expo CLI](https://docs.expo.dev/more/expo-cli/)
- [開発の開始](https://docs.expo.dev/get-started/start-developing/)
