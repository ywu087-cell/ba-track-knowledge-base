# 项目 01 - Amazon Prime Video 观看量预测

## 一句话总结

这个项目用 Amazon Prime Video 电影数据预测每日累计观看时长 (`cvt_per_day`)，最终在 Ridge、Random Forest、Gradient Boosting 中选择了测试集表现最好的 Random Forest。

## 快速目录

1. [项目背景](#项目背景)
2. [数据字段理解](#数据字段理解)
3. [数据清理与特征工程](#数据清理与特征工程)
4. [EDA 发现](#eda-发现)
5. [建模流程](#建模流程)
6. [模型结果](#模型结果)
7. [BA 视角总结](#ba-视角总结)
8. [复习问题](#复习问题)
9. [术语表](#术语表)

## 核心概念索引

- 监督学习 (Supervised Learning)
- 回归问题 (Regression Problem)
- 目标变量 (Target Variable)
- 特征工程 (Feature Engineering)
- 哑变量编码 (Dummy Variables / One-hot Encoding)
- 对数转换 (Log Transformation)
- 伪缺失值 (Pseudo-missing Values)
- 训练集与测试集 (Train/Test Split)
- K 折交叉验证 (K-fold Cross Validation)
- 随机森林 (Random Forest)
- 梯度提升 (Gradient Boosting)
- 平均绝对误差 (MAE)
- 均方根误差 (RMSE)
- 决定系数 (R-squared / R2)

## 项目背景

业务问题是：根据电影本身的信息、外部评分、首页展示位置、类型、内容供应方等字段，预测电影在平台上的每日累计观看时长 (`cvt_per_day`)。

因为 `cvt_per_day` 是连续数值，所以这是一个监督学习中的回归问题 (regression problem)，不是分类问题。

相关 notebook：

- [amazon_prime_video_prediction.ipynb](amazon_prime_video_prediction.ipynb)
- [NOTEBOOK.md](NOTEBOOK.md)：从 Colab 导出的详细复习版，保留主要代码、关键输出和解释。

## 数据字段理解

| 字段 | 含义 | 建模处理 |
| --- | --- | --- |
| `video_id` | 电影唯一编号 | 删除，不作为预测特征 |
| `cvt_per_day` | 每日累计观看时长 | 原始目标变量 |
| `log_cvt_per_day` | 对 `cvt_per_day` 做 log 转换 | 最终目标变量 |
| `weighted_categorical_position` | 首页纵向/分类位置 | 保留 |
| `weighted_horizontal_position` | 首页横向位置 | 保留 |
| `import_id` | 内容合作方 | One-hot encoding |
| `release_year` | 发行年份 | 保留 |
| `genres` | 电影类型，可能有多个 | 拆成 genre dummy |
| `imdb_votes` | IMDb 投票数 | 转成 `log_imdb_votes` |
| `budget` | 电影预算 | 转成 `log_budget`，并保留 missing indicator |
| `boxoffice` | 票房 | 转成 `log_boxoffice`，并保留 missing indicator |
| `imdb_rating` | IMDb 评分 | 保留 |
| `duration_in_mins` | 电影时长 | 保留 |
| `metacritic_score` | Metacritic 分数 | 保留，并保留 missing indicator |
| `awards` | 奖项类别 | One-hot encoding |
| `mpaa` | MPAA 分级 | One-hot encoding |
| `star_category` | 演员热度分数 | 保留，并保留 missing indicator |

## 数据清理与特征工程

### 1. 缺失值检查

数据中没有显式缺失值，也就是 `isnull().sum()` 都是 0。

但是多个字段存在大量 0 值：

| 字段 | 0 值数量 | 解释 |
| --- | ---: | --- |
| `budget` | 2454 | 很可能表示预算未知 |
| `boxoffice` | 3194 | 很可能表示票房未知 |
| `metacritic_score` | 3012 | 很可能表示评分缺失 |
| `star_category` | 1846 | 很可能表示演员热度缺失 |
| `imdb_votes` | 344 | 可能表示 IMDb 信息不足 |
| `imdb_rating` | 344 | 可能表示 IMDb 信息不足 |

因此，这些 0 值不直接当成普通数值理解，而是额外创建 missing indicators：

- `budget_missing`
- `boxoffice_missing`
- `metacritic_missing`
- `star_category_missing`

### 2. 对数转换

`cvt_per_day`、`budget`、`boxoffice`、`imdb_votes` 都明显右偏，因此使用 `np.log1p()` 做对数转换：

- `log_cvt_per_day`
- `log_budget`
- `log_boxoffice`
- `log_imdb_votes`

这样可以降低极端值对模型的影响。

### 3. 类别变量编码

模型不能直接读取文字字段，所以做了两类转换：

- `genres`：用 `str.get_dummies(sep=',')` 拆成多个类型哑变量。
- `import_id`、`awards`、`mpaa`：用 `pd.get_dummies()` 转成 one-hot variables。

### 4. 删除不适合进入模型的字段

建模时删除：

- `video_id`：只是 ID，没有业务预测意义。
- `cvt_per_day`：原始目标变量，不能作为特征。
- `log_cvt_per_day`：最终目标变量，不能放进 X。
- `genres`：原始文字列，已经被 genre dummy 替代。
- `budget`、`boxoffice`、`imdb_votes`：保留 log 版本，避免重复。
- `popularity_group`：EDA 临时分组变量，不进入模型。

## EDA 发现

### 1. 目标变量明显右偏

原始 `cvt_per_day` 中，大部分电影集中在较低观看量区域，少数热门电影观看量很高。对数转换后，分布更接近稳定形态，更适合建模。

### 2. 外部热度比评分更有预测力

和 `log_cvt_per_day` 的相关性中：

- `log_imdb_votes` 相关性较高。
- `imdb_rating` 相关性较弱。

这说明“有多少人关注/投票”比“评分高不高”更能解释平台观看量。

### 3. 首页位置有明显影响

`weighted_categorical_position` 和 `weighted_horizontal_position` 与观看量呈负相关。位置数字越大，通常表示展示位置越靠后，观看量越低。

### 4. Metadata 缺失本身是重要信号

`budget_missing`、`boxoffice_missing`、`metacritic_missing`、`star_category_missing` 都和观看量负相关。也就是说，缺少外部信息的电影通常表现更弱。

### 5. 内容合作方存在差异

不同 `import_id` 的电影在观看量上有明显差异，说明内容来源/版权方可能影响平台表现。

## 建模流程

建模流程如下：

1. 定义目标变量：`y = TV_df['log_cvt_per_day']`
2. 删除无意义字段和目标泄漏字段。
3. 加入 genre dummy。
4. 对 `import_id`、`awards`、`mpaa` 做 one-hot encoding。
5. 使用 `train_test_split` 划分训练集和测试集。
6. 在训练集上做 5-fold cross validation。
7. 用测试集评估最终模型表现。

## 模型结果

| 模型 | CV R2 | Test MAE | Test RMSE | Test R2 | 结论 |
| --- | ---: | ---: | ---: | ---: | --- |
| Ridge Regression | 0.442 | 0.888 | 1.162 | 0.482 | baseline，表现一般 |
| Random Forest | 0.547 | 0.774 | 1.024 | 0.598 | 当前最佳模型 |
| Gradient Boosting | 0.523 | 0.806 | 1.054 | 0.574 | 好于 Ridge，但弱于 Random Forest |

最终选择 Random Forest，因为它在测试集上的 R2 最高，同时 MAE 和 RMSE 最低。

## BA 视角总结

这个项目的重点不是只追求最高模型分数，而是回答业务问题：什么因素会影响电影在平台上的消费表现。

从 BA 视角看，最有价值的结论是：

1. 平台展示位置会影响消费表现，位置越靠后，观看量通常越低。
2. 外部热度信号比单纯评分更有用，IMDb votes、boxoffice、star category 更接近市场关注度。
3. Metadata 缺失不是普通技术问题，它本身反映了电影的市场曝光不足。
4. 内容合作方和类型差异可能帮助平台做内容采购、推荐排序和首页资源分配。
5. Random Forest 更适合这个任务，因为观看量不是简单线性关系，而是多个因素共同作用的结果。

## 复习问题

1. 为什么这个项目是回归问题，而不是分类问题？
2. 为什么 `video_id` 不应该进入模型？
3. 为什么不能把 `cvt_per_day` 放进 `X`？
4. 为什么要对 `cvt_per_day` 做 log transformation？
5. 为什么 0 值不一定等于真实的 0？
6. Genre dummy 和 one-hot encoding 的区别是什么？
7. K-fold cross validation 解决了什么问题？
8. 为什么最终选择 Random Forest？
9. Test R2 和 CV R2 分别代表什么？
10. 从业务角度，metadata 缺失为什么也是 insight？

## 术语表

| 术语 | 英文 | 解释 |
| --- | --- | --- |
| 监督学习 | Supervised Learning | 用已有答案的数据训练模型，再预测新数据 |
| 回归问题 | Regression Problem | 预测连续数值的机器学习任务 |
| 目标变量 | Target Variable | 模型要预测的结果 |
| 特征 | Feature | 用来预测目标变量的输入字段 |
| 目标泄漏 | Target Leakage | 把答案或答案的变形放进特征里，导致模型虚假变好 |
| 哑变量 | Dummy Variable | 把类别转成 0/1 数字列 |
| One-hot encoding | One-hot Encoding | 把一个类别字段拆成多个 0/1 列 |
| 对数转换 | Log Transformation | 压缩右偏数据和极端值影响 |
| 随机森林 | Random Forest | 用多棵决策树综合预测的模型 |
| 交叉验证 | Cross Validation | 多次切分数据验证模型稳定性 |
| MAE | Mean Absolute Error | 平均绝对预测误差 |
| RMSE | Root Mean Squared Error | 对大误差更敏感的预测误差 |
| R2 | R-squared | 模型能解释目标变量变化的比例 |
