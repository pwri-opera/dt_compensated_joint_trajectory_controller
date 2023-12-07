# dt_compensated_joint_trajectory_controllers package
[joint_trajectory_controller](https://github.com/ros-controls/ros2_controllers/tree/master/joint_trajectory_controller)をベースにした zx200_ros2 用controller．
無駄時間補償機能を追加するかは未定．


# joint_trajectory_controller との相違点

## gains パラメータの反映時期
オリジナルではcontrollerがspawnされたタイミングでしかgainsがcommand interfaceの計算に反映されなかったが，update関数を実行する度に反映されるように変更．

`your_joint`のpゲインをコマンドラインから変更するコマンドのサンプルは以下：
```bash
ros2 param set /your_controller_name gains.your_joint.p 0.1
```

## set_hold_position関数の動作
velocity, effortをcommand interfaceとして用いる場合，
オリジナルでは軌道の実行失敗等で set_hold_position関数（現在姿勢を保持しようとする関数）が呼ばれても，velocity, effortには直前の値が残り続ける．
そこで，set_hold_position関数が実行されると出力されるvelocity, effortを0にするよう変更．
また，軌道実行の成功時にset_hold_position関数が呼ばれる用に変更．


