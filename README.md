# dt_compensated_joint_trajectory_controllers package
[joint_trajectory_controller](https://github.com/ros-controls/ros2_controllers/tree/master/joint_trajectory_controller)をベースにした zx200_ros2 用controller．
無駄時間補償機能を追加するかは未定．

> **注）command_interfaceとしてposition, accelerationを使う場合の動作は未確認**

## ビルド方法
- ワークスペースの作成（既にwsを作成済の場合は不要．
以下，新規作成するワークスペースの名称を `ros2_ws` と仮定して表記）
  ```bash
  $ cd ~/
  $ mkdir --parents ros2_ws/src
  $ cd ros2_ws
  $ colcon build 
  ```

- ~/ros2_ws/以下にbuild, install, log, srcディレクトリが作成される

- ~/ros2_ws/src 以下にクローンしてビルド
```bash
$ cd ~/ros2_ws/src
$ git clone https://github.com/pwri-opera/dt_compensated_joint_trajectory_controller.git
$ cd ~/ros2_ws
$ colcon build --symlink-install 
$ . install/setup.bash
```

## joint_trajectory_controller との相違点

### gains パラメータの反映時期
オリジナルではcontrollerがspawnされたタイミングでしかgainsがcommand interfaceの計算に反映されなかったが，update関数を実行する度に反映されるように変更．

`your_joint`のpゲインをコマンドラインから変更するコマンドのサンプルは以下：
```bash
ros2 param set /your_controller_name gains.your_joint.p 0.1
```

### set_hold_position関数の動作
velocity, effortをcommand interfaceとして用いる場合，
オリジナルでは軌道の実行失敗等で set_hold_position関数（現在姿勢を保持しようとする関数）が呼ばれても，velocity, effortには直前の値が残り続ける．
そこで，set_hold_position関数が実行されると出力されるvelocity, effortを0にするよう変更．
また，軌道実行の成功時にset_hold_position関数が呼ばれる用に変更．

## パラメータ

- `joints`: コントローラーが使用するジョイントの名前の配列．デフォルトは空の配列．
- `command_joints`: コントローラーが使用するコマンドジョイントの名前の配列．デフォルトは空の配列．
- `command_interfaces`: 要求するコマンドインターフェースの名前の配列．デフォルトは空の配列．有効なオプションは "position", "velocity", "acceleration", "effort"．
- `state_interfaces`: 要求する状態インターフェースの名前の配列．デフォルトは空の配列．有効なオプションは "position", "velocity", "acceleration"．
- `allow_partial_joints_goal`: ジョイントの部分セットを持つゴールが許可されるかどうか．デフォルトはfalse．
- `open_loop_control`: コントローラーをオープンループで動作させるかどうか，つまり，コントローラーの開始時にのみハードウェアの状態を読み取る．これは，ロボットが指令された軌道を正確に追従しない場合に便利．デフォルトはfalse．
- `allow_integration_in_goal_trajectories`: ゴールの位置や速度が指定されていないゴールを受け入れるためのゴール軌道の積分が許可されるかどうか．デフォルトはfalse．
- `state_publish_rate`: コントローラーの状態が公開されるレート．デフォルトは50.0．
- `action_monitor_rate`: ステータスの変更が監視されるレート．デフォルトは20.0．
- `interpolation_method`: 使用する補間のタイプ．デフォルトは "splines"．有効なオプションは "splines"と "none"．
- `allow_nonzero_velocity_at_trajectory_end`: 最後の速度点がゼロでなければゴールが拒否されるかどうか．デフォルトはtrue．
- `gains`: 以下のパラメータを持つジョイントのマップ：
  - `p`: PIDの比例ゲイン．デフォルトは0.0．
  - `i`: PIDの積分ゲイン．デフォルトは0.0．
  - `d`: PIDの微分ゲイン．デフォルトは0.0．
  - `i_clamp`: 積分クランプ．正と負の両方の方向で対称．デフォルトは0.0．
  - `ff_velocity_scale`: 速度のフィードフォワードスケーリング．デフォルトは0.0．
  - `normalize_error`: (非推奨) 位置エラーの正規化を -pi から pi に使用する．デフォルトはfalse．
  - `angle_wraparound`: 巻き戻し可能なジョイント（つまり，連続する）のため．位置エラーを -pi から pi に正規化する．デフォルトはfalse．
- `constraints`:
  - `stopped_velocity_tolerance`: トラジェクトリの終端で制御システムが停止していることを示す速度許容値．デフォルトは0.01．
  - `goal_time`: トラジェクトリのゴールを指令時間の前後で達成するための時間許容値．デフォルトは0.0．
  - `__map_joints`:
    - `trajectory`: 移動中のジョイントごとのトラジェクトリオフセット許容値．デフォルトは0.0．
    - `goal`: ゴール位置でのジョイントごとのトラジェクトリオフセット許容値．デフォルトは0.0．

