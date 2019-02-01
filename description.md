# プログラム詳細
## クラス
強化学習では、agentとenvironmentが相互に
### environment
+ init
+ reset
+ get_reward
+ get_state
+ update_state
  - オイラー法
+ get_svg

### agent
#### DQNAgent
+ get_action_value
  - シーケンスを与えた時に行動の選択肢の価値を返す
  - 初期プログラムでは2要素のベクトルが返され、それぞれが、行動をする価値に相当
  - 価値の高い方の行動をする
+ get_greedy_action
+ reduce_epsilon
+ get_epsilon
+ get_action
+ experience_local
+ experience_global
+ experience
+ update_model

### simulator
+ seq: シーケンス。状態量のログを`STATE_NUM`だけ保存。
+ total_reward: 報酬。各シミュレーションステップの報酬を合計する。

## 実行詳細
+ シミュレーションの結果を経験として保存する
+ 良い経験を再現するように学習する

1. agent, environment の設定

2. シミュレーション初期値の設定

3. 繰り返す(1シミュレーション)
  1. 現在の状態からagentの行動(action)を決める(get_action)
    + train=Trueとすると、一定確率でランダムな入力を与える
  2. actionをもとに、状態`state`とシーケンス`seq`と報酬`total_reward`を更新する
  3. ログ（現在のシーケンス、その行動、行動結果としての報酬、行動後のシーケンス）を保存

4. シミュレーション後の処理
+ 報酬の大きなシミュレーションは良い「経験」として保存する
  1. ログをグローバルな目盛りに移す
    1. 報酬が過去の良い経験ベスト100位以内に入る場合には最小報酬の経験を取り除き、今回の経験を保存する(`experience()`)
    2. 今回の経験が優秀でなくても、一定確率で追加する

5. モデルの更新
  1. 学習用に用いる経験を選択して、バッチファイルを生成する
  2. バッチファイルを整形して、学習用変数を生成する
    + 行動の評価値をベースとする(`targets=self.model.predict(x).data.copy()`)
    + 実際に行動に移された方には、報酬を加える

6. モデルの学習
  1. 順方向伝播`self.model.zerograds()`
  2. 損失関数の取得`self.model()`
  3. 逆方向伝播`loss.backward()`
  4. パラメータ更新`self.optimizer.update()`
