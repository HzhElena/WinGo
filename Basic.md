# Prepare for training
* Data
* Monte carlo Tree
* CNN
* Reinforce Learning
## Data
KGS Server game records
https://u-go.net/gamerecords/
获取用于监督学习的 SGF. 可以在 https://u-go.net/gamerecords 获取 15 年时长的高段位对局数据。你也可以从其它来源下载专业比赛的数据库。
* 预处理 SGF
对你的 SGF 文件进行预处理。这需要 SGF 文件中的所有局面并提取出每一个局面的特征以及记录正确的下一步走子。
然后将这些局面分割成块（chunk）——一块用于测试，其它的都用于训练。这个步骤需要一定时间，而且要是你修改了 features.py 文件中的特征提取步骤，你还需要重新预处理。
```
python main.py preprocess data/kgs-*```

注：这句代码用了通配符，比如说：KGS 目录可以名为 data/kgs-2006-01、data/kgs-2006-02 等等
* 监督学习（策略网络）
使用上面预处理过的 SGF 数据（默认输出目录是 ./processed_data/），你可以训练策略网络。
```
python main.py train processed_data/ --save-file=/tmp/
网络训练好了之后，当前模型会被保存在 --save-file。你可以通过如下代码继续训练这个网络：
```
python main.py train processed_data/ --read-file=/tmp/savedmodel
--save-file=/tmp/savedmodel --epochs=10 --logdir=logs/my_training_run
```
此外，你也可以使用 TensorBoard 跟踪你的训练过程——如果你为每一次运行定义了一个不同的名字（如：logs/my_training_run、logs/my_training_run2），你可以将这些运行彼此重叠起来：
```tensorboard --logdir=logs/```
* 与 MuGo 对弈
MuGo 使用了 GTP 协议，你可以通过任何兼容 GTP 的程序来使用它。要调用原始策略网络，使用如下代码：
```python main.py gtp policy --read-file=/tmp/savedmodel
```
要调用集成了 MCTS 的策略网络版本，使用：
```
python main.py gtp mcts --read-file=/tmp/savedmodel
```
通过 GTP 下棋的一种方式是使用 gogui-display（它有一个兼容 GTP 的 UI）。你可以在 GoGui下载 gogui 工具套件。参见 GoGui了解使用 GTP 的有趣方式。
```
gogui-twogtp -black 'python main.py gtp policy --read-file=/tmp/savedmodel' -white 'gogui-display' -size 19 -komi 7.5 -verbose -auto
```
另一种通过 GTP 玩的方式是对抗 GnuGo，同时还能观看比赛
```
BLACK="gnugo --mode gtp"
WHITE="python main.py gtp policy --read-file=/tmp/savedmodel"
TWOGTP="gogui-twogtp -black \"$BLACK\" -white \"$WHITE\" -games 10 \
-size 19 -alternate -sgffile gnugo"
gogui -size 19 -program "$TWOGTP" -computer-both -auto
```
* 运行单元测试
```
python -m unittest discover tests
```
## Networks 
# AlphaGo 在对弈过程中使用了三个神经网络
* 第一个神经网络是一个速度很慢但很准确的策略网络（policy network）。这个网络被训练用来预测人类的走子（大约 57% 的准确度），它会输出一个可能走子的列表，并且每一种走子方式都对应了一个概率。这个网络为蒙特卡洛树搜索（MCTS）提供了可能的走子起点。这个神经网络很慢的一大原因是它具有很大的规模，这是因为这个神经网络的输入是围棋棋盘上的各种计算成本高昂的属性——气的数量、叫吃、征等等。
* 第二个神经网络也是一个策略网络，它比第一个更小更快，但准确度更低（大约 24%），这个网络并不使用复杂的属性作为输入。一旦到达了当前 MCTS 树的叶节点（leaf node），这个第二个更快的网络就会被用来得到一个棋盘局面的可能走子，并且对这个这个最终局面进行评分。
* 第三个神经网络是一个价值网络：它为棋盘输出一个预期获胜的范围，而不会自己下任何棋。
