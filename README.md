# shiyaya readme

### 强化学习基础知识

https://www.jianshu.com/p/9f113adc0c50   
https://zhuanlan.zhihu.com/p/55366623    
https://spinningup.openai.com/en/latest/spinningup/rl_intro3.html#deriving-the-simplest-policy-gradient    

### 关于强化学习生成的文本语法不是很顺畅的问题？？

- 这个项目中reward 中使用的PPL 这一项 可以借鉴一下，借用一个GPT2模型
- 也可以借用 《Learning to summarize from human feedback 》 这篇中的 KL 项？？


# TextRL

Text generation with reinforcement learning using huggingface's transformer.

## Introduction

This project is trying to use reinforcement learning to adjust text generation results.
It is based on any text-generation model on huggingaface's [transformer](https://github.com/huggingface/transformers) with [PFRL](https://github.com/pfnet/pfrl) and [OpenAI GYM](https://gym.openai.com).

## Example   
[Controllable generation via RL to let Elon Musk speak ill of DOGE
](https://voidful.dev/jupyter/2021/07/25/textrl-elon-musk.html)

before: `i think dogecoin is a great idea.`    
after: `i think dogecoin is a great idea, but I think it is a little overused.`    

## Installation
### pip install
```bash
pip install textrl
```

### Build from source
git clone and cd into this project.
```bash
pip install -e .
```

## Usage
### init agent and environment
```python
from textrl import TextRLEnv,TextRLActor

from transformers import AutoTokenizer, AutoModelWithLMHead  
tokenizer = AutoTokenizer.from_pretrained("any models")  
model = AutoModelWithLMHead.from_pretrained("any models")
model.eval()
```
### setup reward function for environment
* predicted(list[str]): will be the list of predicted token
* finish(bool): it met the end of sentence or not
```python
class MyRLEnv(TextRLEnv):
    def get_reward(self, input_text, predicted_list, finish): # predicted will be the list of predicted token
        if "[UNK]" in predicted_list:
            reward = -1
        else:
            reward = 1
        return reward
```

### prepare for training
* observation_input should be a list of all possible input string for model training
```python
env = MyRLEnv(model, tokenizer, observation_input=observaton_list)
actor = TextRLActor(env,model,tokenizer)
agent = actor.agent_ppo(update_interval=10, minibatch_size=2000, epochs=20)
```

### Train
```python
n_episodes = 1000
max_episode_len = 200 # max sentence length

for i in range(1, n_episodes + 1):
    obs = env.reset()
    R = 0 
    t = 0 
    while True:
        action = agent.act(obs)
        obs, reward, done, pred = env.step(action)
        R += reward
        t += 1
        reset = t == max_episode_len
        agent.observe(obs, reward, done, reset)
        if done or reset:
            break
    if i % 10 == 0:
        print('episode:', i, 'R:', R)
    if i % 50 == 0:
        print('statistics:', agent.get_statistics())
print('Finished.')
```
another way to train
```python
import logging
import sys
logging.basicConfig(level=logging.INFO, stream=sys.stdout, format='')

pfrl.experiments.train_agent_with_evaluation(
    agent,
    env,
    steps=1000,
    eval_n_steps=None,
    eval_n_episodes=1500,       
    train_max_episode_len=50,  
    eval_interval=10000,
    outdir='somewhere', 
)
```

### prediction
```python
agent.load("somewhere/best") # loading the best model
actor.predict("input text")
```

## dump trained model to huggingface's model
```shell
textrl-dump --model ./model_path_before_rl --rl ./rl_path --dump ./output_dir
```
