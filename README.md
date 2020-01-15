#Pendulum v0這是我這次強化學習作業選的主題  __________________Overview=======_____________________Details---------------- Name: Pendulum-v0.- Category: Classic Control.Description-----------------擺錘(Agent)開始於隨機位置，目標是將其無摩擦的擺錘保持站立。以最小的旋轉速度和最少的力保持在角度零（垂直）。Starting State----------有角度和角速度，可記為th和thdot，從-pi到pi的隨機角度，以及介於-1和1之間的隨機速度。Observation-----------包括了cos(th)、sin(th)、thdot三個量，且有最大值和最小值的限制。Num | Observation | Min | Max---- | --- | --- | ---0 | cos(theta) | -1.0 | 1.01 | sin(theta) | -1.0 | 1.02 | theta dot | -8.0 | 8.0 Actions-----------只有一個維度，就是電機的控制力矩，且有最大值和最小值的限制。Num | Action | Min | Max---- | --- | --- | ---0 | Joint effort | -2.0 | 2.0Reward-------------  獎勵的精確方程式        -(theta^2 + 0.1*theta_dt^2 + 0.001*action^2)-  Theta在-pi和pi之間標準化。因此，最低成本為-(pi^2 + 0.1*8^2 + 0.001*2^2) = -16.2736044，最高成本為0。- reward=-costs, costs包含三項，一是theta^2 ，二是0.1*theta_dt^2 ，三是0.001*action^2 。第一項這是對於當前倒立擺與目標位置的角度差的懲罰；第二項表示對於角速度的懲罰，畢竟如果我們在到達目標位置（豎直）之後，如果還有較大的速度的話，就越過去了；第三項是對於輸入力矩的懲罰，我們所使用的力矩越大，懲罰越大，因為力矩×角速度=功率，因此較小的較好。     ______________________________________________#學習歷程___________________________________________從HW3-Cartpole-Keras_RL2.ipynb改寫------------------------------------------- 使用Colab- 應用Keras-rl tensorflow 2.0 版在Gym環境學習Pendulum-v0策略- Wrap env 讓其可以在Colab上看Pendulum-v0的模擬影片###將action 設定改成隨機產生(原本教授設定左和右的動作)```python #action = action_my()action = env.action_space.sample() #學習到的動作，目前的設定是隨機產生```- 安裝Keras-rl tensorflow 2.0 beta0版- 定義神經網路的架構，輸入參數分別是狀態的維度及可執行的行動數量###將model的env.action改寫```python #model = agent(env.observation_space.shape[0], env.action_space.n)model = agent(env.observation_space.shape[0], env.action_space.shape[0])print(model.summary())```###改寫到sarsa fit的時候產生錯誤資訊,IndexError: invalid index to scalar variable.```pythonfrom rl.agents import SARSAAgent, DDPGAgentfrom rl.policy import EpsGreedyQPolicy, BoltzmannQPolicypolicy = BoltzmannQPolicy() #sarsa = SARSAAgent(model = model, policy = policy, nb_actions = 4)sarsa = SARSAAgent(model = model, policy = policy, nb_actions = env.action_space.shape[0]) #sarsa.compile('adam', metrics=['mae'])sarsa.compile('adam', metrics=['mse'])sarsa.fit(env, nb_steps = 50000, visualize = False, verbose = 1)```[文件](https://colab.research.google.com/drive/1Yvt2qoiB-SaByz1T2ZOOF0Q-aWmEiHzD#scrollTo=HxBxTicpK9NZ)#使用DDPG（Deep Deterministic Policy Gradient）使用Actor Critic結構,但是輸出的不是行為的概率,而是具體的行為,用於連續動作(continuous action)的預測. DDPG結合了之前獲得成功的DQN結構,提高了Actor Critic的穩定性和收斂，可以成功的解決在連續動作預測上的學不太到東西的問題，用此方法用在需連續動作預測的擺錘。- 先定義Actor和Critic- 再定義記憶庫Memory- 建立並把Actor和Critic融合在一起最後得到結果皆接近最低成本-16.273604，且用到時間很短，所以視為一個有效率的解法。![結果](/Desktop/PP.jpg)