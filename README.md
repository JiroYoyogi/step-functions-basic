# フロートとステート

文法的な話

## Pass

- データの整形
- ダミー処理
- etc

### Passを追加

Passを3つ追加

- 出力
  - 次のステートに渡す引数
- 変数
  - ステートマシーンで共有したい変数

### 出力と変数をセット

- 1つ目のPassの出力

```
{
  "count": 1,
  "message": "Hello"
}
```

- 2つ目のPassの出力

```
{
  "count": "{% $states.input.count + 1 %}"
}
```

- 1つ目のPassの変数

```
{
  "executionId": "{% $states.context.Execution.Id %}"
}
```

- 実行時に入力を渡す

```
{
  "count": "{% $states.input.count %}"
}
```

- 確認したいリスト

```
- 1つ目の出力と2つ目の出力の違い
- 3つ目の出力と入力が一致してる。何も設定しない場合そのまま入力が出力になる
- 変数はステートの最後でセットされる。それ以降のステートで参照出来る
- ステートマシーン実行時に入力を渡す方法
```

## Choice

- Choiceを追加
- 「成功」と「Fail」に分岐

### Rule #1

- Condition

```
{% ($states.input.count) > (100) %}
```

### Fail

- Error - オプション

```
だめだこりゃ
```

※ Failはステートマシーンが"失敗した"として強制終了。逆に成功はステートマシーンが"成功した"として強制終了

## Map

- 配列を処理
- S3のJSON一覧を取得して加工するなど

### 作業

- Choice・Fail・成功を削除
- Pass（2）の次にMapを配置
- Mapの各項目にPassを配置
- Mapの後段にPassを配置

### 処理モード

インラインを選択

- インライン
  - 前のステートから配列を受け取る
- 分散
  - S3のオブジェクト一覧を取得するなど

### 項目ソース

項目を提供を選択

```
[1,2,3]
```

### 各項目のPassの出力

各項目の数字に +1 する処理

```
{
  "count": "{% $states.input + 1 %}"
}
```

## AWSサービス連携・DynamoDB

### 作業

- DynamoDBを作成
- 2つ目のPassの後にDynamoDB GetItemを配置

### DynamoDBを作成

- テーブル名：`sfn-dev`
- パーティションキー：`id`

### 項目を作成（DynamoDB）

```
{
  "id": {
    "S": "count"
  },
  "value": {
    "N": "100"
  }
}
```

### 引数と出力（SFn）

- 引数

```
{
  "TableName": "sfn-dev",
  "Key": {
    "id": {
      "S": "count"
    }
  }
}
```

- 出力

ここまでの count + DynamoDB から取得した値

```
{
  "count": "{% $states.input.count + $number($states.result.Item.value.N) %}"
}
```

## Parallel

- DynamoDB GetItemの後にParallelを配置
- 左に元々あったPass、右にWaitを配置

### 出力の設定

- Passの出力

```
{
  "name": "パス"
}
```

- Waitの出力

```
{
  "name": "ウェイト"
}
```