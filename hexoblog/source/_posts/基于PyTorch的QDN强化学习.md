---
title: 基于PyTorch的QDN强化学习
img: /images/DQN.jpg
categories: python
tags:
  - pyTorch
  - DQN
abbrlink: 21869
date: 2022-07-30 22:08:11
---

## pytorch 基础

PyTorch是一个基于Torch的Python开源机器学习库，用于自然语言处理等应用程序。 它主要由Facebook的人工智能研究小组开发。Uber的"Pyro"也是使用的这个库。
PyTorch是一个Python包，提供两个高级功能： * 具有强大的GPU加速的张量计算（如NumPy） * 包含自动求导系统的的深度神经网络

```python
import torch
# 创建一个形状为（2，3）的空张量
torch.empty(2,3) 
# 创建一个形状为（2.3）的随机张量，每个值在[0,1]之间
torch.rand(2,3) 
# 创建一个0填充的矩阵，数据类型为long:
x = torch.zeros(5, 3, dtype=torch.long)
# 创建tensor并使用现有数据初始化:
x = torch.tensor([5.5, 3])
# NumPy 转换 torch
a = torch.ones(5)
b = a.numpy()
# numpy 转化为pytorch
import numpy as np
a = np.ones(5)
b = torch.from_numpy(a)
# 与Numpy的reshape类似
x = torch.randn(4, 4)
x = torch.randn(4, 4)
y = x.view(16)
z = x.view(-1, 8)  #  size -1 从其他维度推断
print(x.size(), y.size(), z.size())
torch.Size([4, 4]) torch.Size([16]) torch.Size([2, 8])
# 查看向量大小 
print(x.size())
# 向量x和y的点积
x.dot(y)
对x按元素求正弦值
x.sin()
```

**报错**：RuntimeError: expected scalar type Long but found Int

**解决：**

label=label.to(device).long()
optimizer=torch.optim.SGD(albertBertClassifier.parameters(),lr=0.01,momentum=0.9,weight_decay=1e-4)
momentum:
若当前的梯度方向与累积的历史梯度方向一致，则当前的梯度会被加强，从而这一步下降的幅度更大。若当前的梯度方向与累积的梯度方向不一致，则会减弱当前下降的梯度幅度。
动量的引入就是为了加快学习过程，特别是对于高曲率、小但一致的梯度，或者噪声比较大的梯度能够很好的加快学习过程。动量的主要思想是积累了之前梯度指数级衰减的移动平均（前面的指数加权平均）
weight_decay:
权衰量,用于防止过拟合
**错误：**
ImportError: No module named 'keras_contrib'
**解决：**
pip install git+https://www.github.com/keras-team/keras-contrib.git

nn.MSELoss()

nn.MSELoss是一个比较简单的损失函数，它计算输出和目标间的均方误差
图像可以使用 Pillow, OpenCV
音频可以使用 scipy, librosa
文本可以使用原始Python和Cython来加载，或者使用 NLTK或 SpaCy 处理,中文建议使用lac

## DQN 两大利器

{% asset_img 更新神经网络.png 更新神经网络 %}

通过 NN 预测出Q(s2, a1) 和 Q(s2,a2) 的值, 这就是 Q 估计. 然后我们选取 Q 估计中最大值的动作来换取环境中的奖励 reward. 而 Q 现实中也包含从神经网络分析出来的两个 Q 估计值, 不过这个 Q 估计是针对于下一步在 s' 的估计. 最后再通过刚刚所说的算法更新神经网络中的参数.

{% asset_img DQN.png DQN %}

DQN 有一个记忆库用于学习之前的经历. 在之前的简介影片中提到过, Q learning 是一种 off-policy 离线学习法, 它能学习当前经历着的, 也能学习过去经历过的, 甚至是学习别人的经历. 所以每次 DQN 更新的时候, 我们都可以随机抽取一些之前的经历进行学习. 随机抽取这种做法打乱了经历之间的相关性, 也使得神经网络更新更有效率. 

### 游戏案例：立杆子

 Torch 自家模块, 我们还要导入 Gym 环境库模块

源码：DQN_Reinforcement_learning.py

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np
import gym

# Hyper Parameters
BATCH_SIZE = 32
LR = 0.01                   # learning rate
EPSILON = 0.9               # 最优选择动作百分比 greedy policy
GAMMA = 0.9                 ## 奖励递减参数 reward discount
TARGET_REPLACE_ITER = 100   # Q 现实网络的更新频率 target update frequency
MEMORY_CAPACITY = 2000  # 记忆库大小
env = gym.make('CartPole-v0')   # 立杆子游戏
env = env.unwrapped  #返回基本非包装环境
N_ACTIONS = env.action_space.n      # 杆子能做的动作
N_STATES = env.observation_space.shape[0]       # 杆子能获取的环境信息数
# 环境状态有4个，小车位置，小车速率，杆子角度，杆子角速度
ENV_A_SHAPE = 0 if isinstance(env.action_space.sample(), int) else env.action_space.sample().shape     # to confirm the shape

# DQN 当中建立两个神经网络, 一个是现实网络 (Target Net),
#  一个是估计网络 (Eval Net).
class Net(nn.Module):
    def __init__(self, ):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(N_STATES, 50)
        self.fc1.weight.data.normal_(0, 0.1)   #normal_：正态分布（均值，标准差）
        self.out = nn.Linear(50, N_ACTIONS)
        self.out.weight.data.normal_(0, 0.1)   # initialization

    def forward(self, x):
        x = self.fc1(x)
        x = F.relu(x)
        actions_value = self.out(x)#动作价值
        return actions_value

# 两个 net, 有选动作机制, 有存经历机制, 有学习机制.
class DQN(object):
    def __init__(self):# 建立 target net 和 eval net 还有 memory
        self.eval_net, self.target_net = Net(), Net()

        self.learn_step_counter = 0                                     # 用于 target 更新计时
        self.memory_counter = 0                                         # 记忆库记数 for storing memory
        self.memory = np.zeros((MEMORY_CAPACITY, N_STATES * 2 + 2))    # 初始化记忆库
        self.optimizer = torch.optim.Adam(self.eval_net.parameters(), lr=LR)
        self.loss_func = nn.MSELoss()    # 误差公式

    def choose_action(self, x):# 根据环境观测值选择动作的机制
        x = torch.unsqueeze(torch.FloatTensor(x), 0)#维度增加函數
        # input only one sample
        if np.random.uniform() < EPSILON:   #随机生成0到1且小于0.9，选最优动作
            actions_value = self.eval_net.forward(x)# return the argmax
            action = torch.max(actions_value, 1)[1].data.numpy()
            action = action[0] if ENV_A_SHAPE == 0 else action.reshape(ENV_A_SHAPE)  # return the argmax index
        else:   # 选随机动作
            action = np.random.randint(0, N_ACTIONS)
            action = action if ENV_A_SHAPE == 0 else action.reshape(ENV_A_SHAPE)
        return action

    def store_transition(self, s, a, r, s_): # 存储记忆
        transition = np.hstack((s, [a, r], s_))#hstack():按顺序水平堆叠数组(按列)
        # 如果记忆库满了, 就覆盖老数据
        # replace the old memory with new memory
        index = self.memory_counter % MEMORY_CAPACITY
        self.memory[index, :] = transition
        self.memory_counter += 1

    def learn(self):# target 网络更新   学习记忆库中的记忆
        # target parameter update
        if self.learn_step_counter % TARGET_REPLACE_ITER == 0:
            self.target_net.load_state_dict(self.eval_net.state_dict())#将eval_net的所有参数赋值到target_net参数
        self.learn_step_counter += 1

        # 抽取记忆库中的批数据 sample batch transitions
        sample_index = np.random.choice(MEMORY_CAPACITY, BATCH_SIZE)
        b_memory = self.memory[sample_index, :]
        b_s = torch.FloatTensor(b_memory[:, :N_STATES])
        b_a = torch.LongTensor(b_memory[:, N_STATES:N_STATES+1].astype(int))
        b_r = torch.FloatTensor(b_memory[:, N_STATES+1:N_STATES+2])
        b_s_ = torch.FloatTensor(b_memory[:, -N_STATES:])
        # 针对做过的动作b_a, 来选 q_eval 的值, (q_eval 原本有所有动作的值)
        # q_eval w.r.t the action in experience
        q_eval = self.eval_net(b_s).gather(1, b_a)  # shape (batch, 1)
        q_next = self.target_net(b_s_).detach()     # q_next 不进行反向传递误差, 所以 detach，将tensor中的梯度值扔了，只保留矩阵
        q_target = b_r + GAMMA * q_next.max(1)[0].view(BATCH_SIZE, 1)   #[0]：最大值 [1]:对应索引shape (batch, 1)
        loss = self.loss_func(q_eval, q_target)
        # 计算, 更新 eval net
        self.optimizer.zero_grad()# 清空上一步的残余更新参数值
        loss.backward() # 误差反向传播, 计算参数更新值
        self.optimizer.step()   # 将参数更新值施加到 net 的 parameters 上
# 按照 Qlearning 的形式进行 off-policy 的更新. 我们进行回合制更行,
# 一个回合完了, 进入下一回合. 一直到他们将杆子立起来很久.
dqn = DQN()# 定义 DQN 系统

print('\nCollecting experience...')
for i_episode in range(400):#400个回合
    s = env.reset()
    ep_r = 0
    while True:
        env.render()    # 显示实验动画
        a = dqn.choose_action(s)

        # 选动作, 得到环境反馈
        s_, r ,done,terminated,info = env.step(a)

        # 修改 reward, 使 DQN 快速学习
        x, x_dot, theta, theta_dot = s_
        r1 = (env.x_threshold - abs(x)) / env.x_threshold - 0.8
        r2 = (env.theta_threshold_radians - abs(theta)) / env.theta_threshold_radians - 0.5
        r = r1 + r2
        # 存记忆
        dqn.store_transition(s, a, r, s_)

        ep_r += r
        if dqn.memory_counter > MEMORY_CAPACITY:
            dqn.learn()# 记忆库满了就进行学习
            if done:# 如果回合结束, 进入下回合
                print('Ep: ', i_episode,
                      '| Ep_r: ', round(ep_r, 2))

        if done:
            break
        s = s_
```

### 效果展示

{% asset_img 立杆子.gif 立杆子 %}



