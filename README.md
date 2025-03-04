# Gym-μRTS with Stable-Baselines3/PyTorch

This repo contains an attempt to reproduce Gridnet PPO with invalid action masking algorithm to play μRTS using [Stable-Baselines3](https://github.com/DLR-RM/stable-baselines3) library. Apart from reproducibility, this might open access to a diverse set of well tested algorithms, and toolings for training, evaluations, and more.

Original paper: [Gym-μRTS: Toward Affordable Deep Reinforcement Learning Research in Real-time Strategy Games](https://arxiv.org/abs/2105.13807).

Original code: [gym-microrts-paper](https://github.com/vwxyzjn/gym-microrts-paper).

![demo.gif](https://github.com/vwxyzjn/gym-microrts/raw/master/static/fullgame.gif)

## Install

Prerequisites:
* Python 3.7+
* Java 8.0+
* FFmpeg (for video capturing)

```
git clone https://github.com/kachayev/gym-microrts-paper-sb3
cd gym-microrts-paper-sb3
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Note that I use newer version of `gym-microrts` compared to the one that was originally used for the paper.

## Training

To traing an agent:

```
$ python ppo_gridnet_diverse_encode_decode_sb3.py
```

If everything is setup correctly, you'll see typicall SB3 verbose logging:

```
Using cpu device
---------------------------------
| rollout/           |          |
|    ep_len_mean     | 2e+03    |
|    ep_rew_mean     | 0.0      |
| time/              |          |
|    fps             | 179      |
|    iterations      | 1        |
|    time_elapsed    | 11       |
|    total_timesteps | 2048     |
---------------------------------
------------------------------------------
| rollout/                |              |
|    ep_len_mean          | 1.72e+03     |
|    ep_rew_mean          | -5.0         |
| time/                   |              |
|    fps                  | 55           |
|    iterations           | 2            |
|    time_elapsed         | 74           |
|    total_timesteps      | 4096         |
| train/                  |              |
|    approx_kl            | 0.0056759235 |
|    clip_fraction        | 0.0861       |
|    clip_range           | 0.2          |
|    entropy_loss         | -5.65        |
|    explained_variance   | 0.412        |
|    learning_rate        | 0.0003       |
|    loss                 | -0.024       |
|    n_updates            | 10           |
|    policy_gradient_loss | -0.00451     |
|    value_loss           | 0.00413      |
------------------------------------------
```

As soon as correctness of the implementation is verified, I will provide details on how to use RL Baselines3 Zoo for training and evaluations.

## Implementational Caveats

A few notes / pain points regarding the implementation of the alrogithms, and the process of integrating it with stable-baselines3:

* Gym does not ship a space for "array of multidiscrete" use case (let's be honest, it's not very common). But it gives an option for defining your space when necessary. A new space, when defined, is not easy to integrate into SB3. In a few different places SB3 raises `NotImplementedError` facing unknown space ([example 1](https://github.com/DLR-RM/stable-baselines3/blob/df6f9de8f46509dad47e6d2e5620aa993b0fc883/stable_baselines3/common/distributions.py#L644), [example 2](https://github.com/DLR-RM/stable-baselines3/blob/df6f9de8f46509dad47e6d2e5620aa993b0fc883/stable_baselines3/common/preprocessing.py#L183)).
* Seems like switching to fully rolled out `MutliDiscrete` space definition has a significant performance penalty. Still investigating if this can be improved.
* Invalid masking is implemented by passing masks into observations from the wrapper (the observation space is replaced with `gym.spaces.Dict` to hold both observations and masks). By doing it this way, masks are now available for policy, and fit rollout buffer layout. Masking is implemented by setting logits into `-inf` (or to a rather small number).

Look for `xxx(hack)` comments in the code for more details.