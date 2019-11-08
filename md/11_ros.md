# ロボットシステム学第11回

上田 隆一

2019年12月6日@千葉工業大学

## 今日の内容

* ROS


## ROS: robot operating system

* ロボットのソフトウェアコンポーネントを作って
  動作させるためのフレームワーク/ミドルウェア
  * OSでは無い
* 発祥: 2000年代後半、Willow Garage社
* BSDライセンス
* Linux（Ubuntu）上での動作がデフォルト
* サイト
  * 公式ページ: http://www.ros.org/
  * マニュアル等: http://wiki.ros.org/ja


## どんなものか

* 本体: プロセス間通信をつかさどる
  * プロセス同士をpublish-subscribeモデルやサービスでつなぐ
  * XML-RPC等を利用
  * 通信するデータに型
* 周辺
  * ビルドシステム、パッケージ管理、テストツール、・・・


<span style="color:red">と書いてもよくわからんのでこちらで動かしてみます</span>



## デモ

* カメラの画像をブラウザから見る
* 手順
  * 1: ROS・ワークスペースのセットアップ（来週）
  * 2: 必要なパッケージのダウンロードとビルド
    ```bash
    $ sudo apt install ros-kinetic-cv-camera
    $ sudo apt install ros-kinetic-cv-bridge
    $ cd ~/catkin_ws/src
    $ git clone git@github.com:RobotWebTools/mjpeg_server.git
    $ cd ..
    $ catkin_make
    ```
  * 3: 見る
    ```
    $ roscore &
    $ rosrun cv_camera cv_camera_node &
    $ rosrun mjpeg_server mjpeg_server
    ブラウザでhttp://<IPアドレス>:8080/stream?topic=/cv_camera/image_raw
    ```

## さらに便利に使う

* ROS化されているキラーコンテンツ
  * slam_gmapping, ナビゲーションメタパッケージ
    * 地図生成（次のページにデモ）、位置推定、経路生成
  * MoveIt!
    * 腕の動作計画 腕先の位置を入力→関節角を計算（逆運動学）
  * その他、様々なハード・ソフトがROS化

## ROSを使ったSLAMの様子

* https://www.youtube.com/embed/b2kYQ11PUSI"
  * ポイント: SLAMのコードは書いてない
    * aptでSLAM（Gmapping）のコードが入る
    * デッドレコニングのコードをちょっと書いただけ

## ROSを使ったナビゲーションの様子

* https://youtu.be/RpPcmyXOcr4
  * SLAMをした後のナビゲーション
    * 自己位置推定、障害物回避、...
    * 足回りもROSのパッケージ化

## ROSのインストール

* Ubuntu 16.04にインストールして使用
  * 次のリポジトリにインストーラ
    * https://github.com/ryuichiueda/ros_setup_scripts_Ubuntu16.04_server
      * 中にあるシェルスクリプトをstep0.bash, step1.bash, locale.ja.bashと実行すればOK
* 講義ではインストール済みのイメージファイルを利用
  * 参考: [昨年の講義資料](https://lab.ueda.tech/?presenpress=%E3%83%AD%E3%83%9C%E3%83%83%E3%83%88%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E5%AD%A62016%E7%AC%AC12%E5%9B%9E#/15)
  * Raspberry Pi 3B+ を使用している方は[上田研の齋藤篤志さんのブログ](https://www.asrobot.me/entry/2018/07/11/001603/)を参照
## 動作確認

* `roscore`
  * ROSの基盤となるプログラムが立ち上がる
  * Ctrl+cで出る

```bash
$ roscore
（略）
started roslaunch server http://localhost:39310/
ros_comm version 1.12.6

SUMMARY
========

PARAMETERS
* /rosdistro: kinetic
* /rosversion: 1.12.6

NODES

auto-starting new master
process[master]: started with pid [1439]
ROS_MASTER_URI=http://localhost:11311/

setting /run_id to b749a100-d0dc-11e5-a506-b827eb17cb96
process[rosout-1]: started with pid [1452]
started core service [/rosout]
```

## ROSのノード

* プログラムのプロセス一つ一つが「ノード」と呼ばれる
* ノードの例
  * cv_cameraとmjpeg_serverを立ち上げる（roscoreは立ち上げておく）

  ```bash
  $ rosrun cv_camera cv_camera_node 
  $ rosrun mjpeg_server mjpeg_server 
  ```

  * ノードの確認

  ```bash
  $ rosnode list
  /cv_camera
  /mjpeg_server
  /rosout
  ```

* ディレクトリのように管理されている

## トピック・メッセージ

* 今度はrostopic listと打ってみる
  * データをやり取りする口（トピック）が表示される

  ```bash
  $ rostopic list
  /cv_camera/camera_info
  /cv_camera/image_raw
  /rosout
  /rosout_agg
  ```

  * トピックからデータを取り出す（先ほどもやりました）

    ```bash
    $ rostopic echo /cv_camera/image_raw
    ```

    * このデータは「メッセージ」と呼ばれる

## パブリッシャ・サブスクライバ

* 各ノードがトピックを通じてメッセージを融通することで全体として仕事を行う
* mjpeg_serverは/cv_camera/image_rawからカメラ画像を取得して、ブラウザに画像を配信
* データを出す側が**パブリッシャ**
* データを受け取る側が**サブスクライバ**
* この構造でサブスクライバ側の柔軟な組み換えが可能に
  * ブラウザに配信するノード、顔検出をするノード、mp4に変換するノード・・・

## ROSプログラミングの準備

* パブリッシャ、サブスクライバを作ってみましょう
* その前に・・・
  * 「ワークスペース」（作業場）を作る
  * 講義で使うイメージでは作成済み

* 手順

```bash
$ cd
$ mkdir -p catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace 
Creating symlink "/home/ubuntu/catkin_ws/src/CMakeLists.txt" pointing to "/opt/ros/kinetic/share/catkin/cmake/toplevel.cmake"
$ ls
CMakeLists.txt
```

* .bashrcの末尾に以下を記述

```bash
source /opt/ros/kinetic/setup.bash          #これは元からある
source ~/catkin_ws/devel/setup.bash         #ここから3行追加
export ROS_MASTER_URI=http://localhost:11311
export ROS_HOSTNAME=localhost
```

* 環境のビルド

```bash
$ cd ~/catkin_ws
$ catkin_make
$ source ~/.bashrc
```


* 確認
  * ROS_PACKAGE_PATHにcatkin_ws/srcがセットされているはず

```bash
$ echo $ROS_PACKAGE_PATH
/home/ubuntu/catkin_ws/src:/opt/ros/kinetic/share
```

## パッケージを作る

* パッケージ: いくつかのノードを含んだ一単位
* パッケージの生成
  * `catkin_create_pkg <作るパッケージの名前> [使用するライブラリ...]`
  * `rospy`: Pythonでノードを作るときに使用
  * パッケージを作ったら下に`scripts`というディレクトリを作成
    * ここにノードとなるプログラムを置く

```bash
$ cd ~/catkin_ws/src
$ catkin_create_pkg mypkg rospy
 Created file mypkg/package.xml
 Created file mypkg/CMakeLists.txt
 Created folder mypkg/src
 Successfully created files in /home/ubuntu/catkin_ws/src/mypkg. Please adjust the values in package.xml.
$ cd mypkg/
$ mkdir scripts
$ cd scripts/
```

## パブリッシャを作る

* 次のようなプログラム（count.py）を書いてみましょう
* ノード名が「count」、パブリッシャが「count_up」
* rospy.Publisherを作って定期的にデータを投げる
  * count_upというトピックに、型はInt32で（バッファとなるキューのサイズは1）

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

if __name__ == '__main__':
    rospy.init_node('count')
    pub = rospy.Publisher('count_up', Int32, queue_size=1)
    rate = rospy.Rate(10)
    n = 0
    while not rospy.is_shutdown():
        n += 1
        pub.publish(n)
        rate.sleep()
```

## ノードの実行

注意: あらかじめroscoreを立ち上げておきましょう。

```bash
$ rosrun mypkg count.py
```

* rosnode listとrostopic listでノードとトピックの確認を
* 次にrostopic echoでcount_upからデータを取り出してみましょう

```
$ rostopic echo /count_up 
data: 1430
---
data: 1431
---
data: 1432
---
data: 1433
...
```

## サブスクライバを作る

* 次のようなtwice.pyを作る
* rospy.Subscriberを使う
  * count_upという名前のトピックを購読する
  * 型はInt32
  * データを受け取ったときにcbという関数で処理
    * コールバック関数

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

def cb(message):
    rospy.loginfo(message.data*2)

if __name__ == '__main__':
    rospy.init_node('twice')
    sub = rospy.Subscriber('count_up', Int32, cb)
    rospy.spin()
```

## twice.pyの実行

* roscore、rosrun mypkg count.pyを事前に実行しておく
* 二倍になった数字が端末上に表示される
  * 非同期なので欠ける可能性があることに注意
  * 途中で切ってまた立ち上げても再開

```bash
$ rosrun mypkg twice.py
[INFO] [1484659840.762425]: 4
[INFO] [1484659840.862150]: 6
[INFO] [1484659840.961791]: 8
[INFO] [1484659841.062162]: 10
...
```

## パブリッシャとサブスクライバの同居

* twice.pyにパブリッシャを追加

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

n = 0

def cb(message):
    global n
    n = message.data*2

if __name__ == '__main__': 
    rospy.init_node('twice')
    sub = rospy.Subscriber('count_up', Int32, cb) 
    pub = rospy.Publisher('twice', Int32, queue_size=1) 
    rate = rospy.Rate(10)
    while not rospy.is_shutdown():
        pub.publish(n)
        rate.sleep()
```

## 実行

* ノードを立ち上げて`rostopic echo /twice`でトピックとしてデータを得る

```bash
$ rostopic echo /twice
data: 1050
---
data: 1052
---
data: 1054
---
data: 1056
...
```

## その他

* 型
  * `std_msgs`で定義されているものの他にもたくさん
    * `rosmsg list`を打ってみましょう
  * 自分で定義することも可能
* サービス
  * あるノードが他のノードのプログラムを実行
  * トピックと異なり、処理が終わるまでウェイトがかかる（同期処理）
* actionlib
  * サービスと同じくノード間でプログラムを呼び出す仕組み
    * 中断したり途中経過をみたりすることが可能
    * 時間のかかる処理の実装に利用される
* package.xml
  * パッケージの情報が書かれる
  * メンテナの名前、ライセンス、他のパッケージの依存関係等、配布に必要な情報
* CMakeLists.txt
  * ビルド情報
  * 自分で型を作る、C++でノードを作る等、講義の内容より複雑なことをするときには手を入れる必要がある
* 参考
  * [小倉: ROSではじめるロボットプログラミング, 工学社, 2015.](https://www.kohgakusha.co.jp/books/detail/978-4-7775-1901-9)
  * [上田: Raspberry Piで学ぶ　ROSロボット入門, 日経BP, 2017.](http://ec.nikkeibp.co.jp/item/books/261040.html)
