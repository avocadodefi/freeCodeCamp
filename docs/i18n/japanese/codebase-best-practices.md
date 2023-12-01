# コードベースのベストプラクティス

## コンポーネントのスタイリング

コンポーネントのスタイリングには、私たちの [design style guide](https://design-style-guide.freecodecamp.org/) の使用を推奨します。

色は [`variable.css`](/client/src/components/layouts/variables.css), で定義されており、フォントは [`fonts.css`](/client/src/components/layouts/fonts.css). にあります。

新しい変数/トークンを色に追加することについては、強く意見を持っています。慎重な研究の後、色はfreeCodeCampのブランドアイデンティティ、開発者体験、およびアクセシビリティを尊重するために選ばれました。

!important キーワードは、一部のケース（例えばアクセシビリティの懸念事項）で値を上書きするために使用される場合があります。将来のリファクタリングで削除されないように、問題を説明するコメントを追加してください。

RTLサポート
当社は、この方向で読まれる言語のために、コードベースで右から左（RTL）レイアウトのサポートを目指しています。これを行うには、コンポーネントのスタイリング方法に注意する必要があります。以下は、守るべき簡単なルールのいくつかです：

float プロパティを使用しない
代わりにFlexboxやGridレイアウトを使用すると、既にRTLサポートが組み込まれており、メンテナンスやレビューが容易になります。
margin と padding を使用する際に方向を定義しない：padding-right や margin-left を使用するのは無害に見えるかもしれませんが、レイアウトがRTLに変更されたときにこれらの方向が反映されないため、RTLファイルにそれらの対向値を追加するとコードベースのメンテナンスが難しくなります。
代わりに論理的なプロパティを使用する：padding-inline-end や margin-inline-start を使用することで同じスペースを追加でき、RTLレイアウトに関して心配する必要がなくなります。これらは行の開始と終了に従いますので、RTLファイルに余分な値を追加する必要もなく、二つのファイルで同じ値を変更する必要があることを覚えておく必要もありません。
font-family に !important を使用しない：RTLレイアウトはLTRレイアウトと比べて異なるフォントを使用するため、font-family プロパティに !important を追加するとRTLレイアウトにも影響します。

## General JavaScript

In most cases, our [linter](how-to-setup-freecodecamp-locally.md#follow-these-steps-to-get-your-development-environment-ready) will warn of any formatting which goes against this codebase's preferred practice.

It is encouraged to use functional components over class-based components.

## Specific TypeScript

### Migrating a JavaScript File to TypeScript

#### Git のファイル履歴を保持する

ファイルを <filename>.js から <filename>.ts（または .tsx）に変更すると、元のファイルが削除されて新しいファイルが作成されることがありますし、別の場合には単にファイル名が変更されるだけです - Gitの観点から見ると。理想的には、ファイルの履歴を保持したいものです。

これを実現するための最良の方法は以下の通りです：

1. ファイル名を変更する
2. フラグ `--no-verify` でコミットして、Husky がリントエラーについて不平を言うことを防ぐ
3. 別のコミットで、移行のために TypeScript にリファクタリングする

> [!NOTE] VScode 等のエディターは、ファイルが削除され新しいファイルが作成されたことを表示する可能性があります。 `git add .` に CLI を使用すると、VSCode はファイル名が変更されたものとしてステージに表示します。

### Naming Conventions

#### インターフェースと型

For the most part, it is encouraged to use interface declarations over type declarations.

React Component Props - suffix with `Props`

```typescript
interface MyComponentProps {}
// type MyComponentProps = {};
const MyComponent = (props: MyComponentProps) => {};
```

React Stateful Components - suffix with `State`

```typescript
interface MyComponentState {}
// type MyComponentState = {};
class MyComponent extends Component<MyComponentProps, MyComponentState> {}
```

Default - object name in PascalCase

```typescript
interface MyObject {}
// type MyObject = {};
const myObject: MyObject = {};
```

<!-- #### Redux Actions -->

<!-- TODO: Once refactored to TS, showcase naming convention for Reducers/Actions and how to type dispatch funcs -->

## Redux

### Action Definitions

```typescript
enum AppActionTypes = {
  actionFunction = 'actionFunction'
}

export const actionFunction = (
  arg: Arg
): ReducerPayload<AppActionTypes.actionFunction> => ({
  type: AppActionTypes.actionFunction,
  payload: arg
});
```

### How to Reduce

```typescript
// Base reducer action without payload
type ReducerBase<T> = { type: T };
// Logic for handling optional payloads
type ReducerPayload<T extends AppActionTypes> =
  T extends AppActionTypes.actionFunction
    ? ReducerBase<T> & {
        payload: AppState['property'];
      }
    : ReducerBase<T>;

// Switch reducer exported to Redux combineReducers
export const reducer = (
  state: AppState = initialState,
  action: ReducerPayload<AppActionTypes>
): AppState => {
  switch (action.type) {
    case AppActionTypes.actionFunction:
      return { ...state, property: action.payload };
    default:
      return state;
  }
};
```

### How to Dispatch

Within a component, import the actions and selectors needed.

```tsx
// Add type definition
interface MyComponentProps {
  actionFunction: typeof actionFunction;
}
// Connect to Redux store
const mapDispatchToProps = {
  actionFunction
};
// Example React Component connected to store
const MyComponent = ({ actionFunction }: MyComponentProps): JSX.Element => {
  const handleClick = () => {
    // Dispatch function
    actionFunction();
  };
  return <button onClick={handleClick}>freeCodeCamp is awesome!</button>;
};

export default connect(null, mapDispatchToProps)(MyComponent);
```

<!-- ### Redux Types File -->
<!-- The types associated with the Redux store state are located in `client/src/redux/types.ts`... -->

## API

### Testing

The `api/` tests are split into two parts:

1. Unit tests
2. Integration tests

#### Unit Tests

Unit tests isolate a single function or component. The tests do not need mocking, but will require fixtures.

The unit tests are located in a new file adjacent to the file exporting that is being tested:

```text
api/
├── src/
│   ├── utils.ts
│   ├── utils.test.ts
```

####統合テスト

統合テストは、API全体をテストします。テストにはモッキングが必要であり、データベースのシードデータと認証用の方法を超えるフィクスチャは必要ありません。

通常、各統合テストファイルは特定のルートに直接関連しています。統合テストは api/tests/ ディレクトリにあります：
```text
api/
├── tests/
│   ├── settings.ts
```

## Further Literature

- [TypeScript Docs](https://www.typescriptlang.org/docs/)
- [TypeScript with React CheatSheet](https://github.com/typescript-cheatsheets/react#readme)
