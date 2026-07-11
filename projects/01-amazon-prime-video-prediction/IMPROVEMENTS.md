# Amazon Prime Video 项目改进指南

## 一句话总结

这一页把老师版本中值得补充的步骤整理为可独立运行的改进实验：识别伪缺失、合并稀有类型、正确标准化、调优 Random Forest，并把模型结果转成业务可用的特征重要性。

## 快速目录

1. [使用原则](#使用原则)
2. [0 值比例与伪缺失](#0-值比例与伪缺失)
3. [完整的改进版特征准备](#完整的改进版特征准备)
4. [稀有 Genre 合并](#稀有-genre-合并)
5. [年份分箱](#年份分箱)
6. [正确的标准化与 Ridge 基准模型](#正确的标准化与-ridge-基准模型)
7. [GridSearchCV 调优 Random Forest](#gridsearchcv-调优-random-forest)
8. [模型比较与特征重要性](#模型比较与特征重要性)
9. [BA 视角与注意事项](#ba-视角与注意事项)

## 使用原则

这是一份 **improvement experiment（改进实验）**，建议在原始 Colab notebook 的副本或新 section 中运行，不要覆盖已经记录的 baseline 结果。

原始项目的优势是：保留了 `budget_missing` 等 missing indicator，以研究“信息缺失本身是否是业务信号”。本指南保留这个思路，同时把 0 值转成缺失值并填补，令模型能够同时利用：

- 数值本身，例如预算金额；
- 原始资料是否缺失，例如 `budget_missing = 1`。

以下代码默认已经读入原始数据为 `TV_df`，并且字段名与项目 notebook 一致。

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
```

## 0 值比例与伪缺失

先看 0 值占每一列的比例，而不是只看数量。`True` / `False` 的平均值会自动变成 `1` / `0` 的平均值，因此得到的就是比例。

```python
zero_ratio = (TV_df == 0).mean().sort_values(ascending=False)
print(zero_ratio)
```

例如 `boxoffice` 的结果若为 `0.756`，表示约 75.6% 的电影票房是 0。结合业务语义，这些 0 往往更像“未知或未收录”，而不是电影真实没有票房。

## 完整的改进版特征准备

这段代码完成四件事：复制原始数据、防止改坏 baseline、创建 missing indicators、用中位数填补伪缺失并对强右偏变量取对数。

```python
# 使用副本做改进实验，不改变原本的 TV_df
model_df = TV_df.copy()

# 这些字段的 0 在本项目中更可能代表“信息未知”
pseudo_missing_cols = [
    'budget',
    'boxoffice',
    'metacritic_score',
    'star_category',
    'imdb_votes',
    'imdb_rating'
]

missing_indicator_map = {
    'budget': 'budget_missing',
    'boxoffice': 'boxoffice_missing',
    'metacritic_score': 'metacritic_missing',
    'star_category': 'star_category_missing',
    'imdb_votes': 'imdb_votes_missing',
    'imdb_rating': 'imdb_rating_missing'
}

# 1 = 原始资料为 0 / 未知；0 = 原始资料存在
for col in pseudo_missing_cols:
    indicator_col = missing_indicator_map[col]
    model_df[indicator_col] = (model_df[col] == 0).astype(int)

    # 把伪缺失的 0 转成真正的缺失值，再用中位数填补
    model_df[col] = model_df[col].replace(0, np.nan)
    model_df[col] = model_df[col].fillna(model_df[col].median())

# 对目标变量和明显右偏的连续变量取对数
model_df['log_cvt_per_day'] = np.log1p(model_df['cvt_per_day'])
model_df['log_budget'] = np.log1p(model_df['budget'])
model_df['log_boxoffice'] = np.log1p(model_df['boxoffice'])
model_df['log_imdb_votes'] = np.log1p(model_df['imdb_votes'])

# 建模目标：预测 log 后的每日观看时长
y = model_df['log_cvt_per_day']

# 原始强右偏列已经由 log 版本替代；目标列及 ID 不能进入 X
drop_cols = [
    'video_id',
    'cvt_per_day',
    'log_cvt_per_day',
    'genres',
    'budget',
    'boxoffice',
    'imdb_votes'
]

X = model_df.drop(columns=drop_cols)
```

为什么使用中位数（median）而不是均值（mean）？预算、票房和投票数仍可能有极端大的影片；中位数更不容易被少数爆款拉高。这里保留 missing indicator，因此填补不会抹掉“这条资料原本缺失”的业务信息。

## 稀有 Genre 合并

一部电影可能有多个 genre，所以用 `str.get_dummies(sep=',')`，而不是普通的 `pd.get_dummies()`。然后将样本太少的类型合并成 `genre_Misc`，避免模型把少数影片的偶然表现当作规律。

```python
# 将多标签 genres 拆成多个 0/1 列
genre_dummies = model_df['genres'].str.get_dummies(sep=',').add_prefix('genre_')

# 先查看每个类型出现多少次，再决定稀有类型阈值
genre_counts = genre_dummies.sum().sort_values()
print(genre_counts.head(10))

# 本数据约有 4,226 部电影；这里先将出现少于 50 次的类型合并
rare_genres = genre_counts[genre_counts < 50].index.tolist()

if rare_genres:
    # 某部电影只要包含任意一个稀有类型，genre_Misc 就为 1
    genre_dummies['genre_Misc'] = genre_dummies[rare_genres].max(axis=1)
    genre_dummies = genre_dummies.drop(columns=rare_genres)

# 将处理后的 genre 特征加回模型表
X = pd.concat([X, genre_dummies], axis=1)
```

`50` 不是固定真理，而是一个可测试的起点。若某个稀有 genre 有明确业务意义，例如平台需要专门评估 `Anime` 内容，则可以保留，不合并。

## 年份分箱

老师将 `release_year` 分成若干年代组，再做 dummy variables。这样做能让线性模型学习“不同年代整体表现不同”，而不是假设年份每加一年，观看量都按同一幅度变化。

对于本项目最终选择的 Random Forest，保留原始 `release_year` 通常已经足够，因为树模型会自己寻找合适的切分点。因此年份分箱是 **Ridge/Lasso 的可选实验**，不要和原始年份同时放进同一个模型。

```python
# 可选：仅用于 Ridge / Lasso 的年份分箱版本
bin_year = [1916, 1974, 1991, 2001, 2006, 2008, 2010, 2012, 2013, 2014, 2017]
year_range = [
    '1916-1974', '1974-1991', '1991-2001', '2001-2006',
    '2006-2008', '2008-2010', '2010-2012', '2012-2013',
    '2013-2014', '2014-2017'
]

year_bin = pd.cut(
    model_df['release_year'],
    bins=bin_year,
    labels=year_range,
    include_lowest=True
)
year_dummies = pd.get_dummies(year_bin, prefix='release_year')

# 用年份组替换原始年份；不要同时保留两种版本
X_year_binned = X.drop(columns='release_year')
X_year_binned = pd.concat([X_year_binned, year_dummies], axis=1)
```

## 文字类别的 One-hot Encoding

`import_id`、`awards` 和 `mpaa` 每部电影只属于一个类别，因此可直接 one-hot encoding。完成后，模型输入必须全部是数值。

```python
categorical_cols = ['import_id', 'awards', 'mpaa']
X = pd.get_dummies(X, columns=categorical_cols, dtype=int)

# 检查是否仍有文字类型；正常结果应是空 Index
print(X.select_dtypes(include='object').columns)
```

## 正确的标准化与 Ridge 基准模型

标准化（Standardization）只在 Ridge / Lasso 等对尺度敏感的模型中需要；Random Forest 不需要。关键顺序是：**先切分数据，再只用训练集拟合 scaler**。这样测试集的信息不会提前泄漏给训练流程。

```python
# 这里使用原始 release_year。若要测试年份分箱，则将 X 改成 X_year_binned。
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.25,
    random_state=42
)

# 只缩放连续数值列；0/1 dummy variables 不需要缩放
continuous_cols = [
    'weighted_categorical_position',
    'weighted_horizontal_position',
    'release_year',
    'imdb_rating',
    'duration_in_mins',
    'metacritic_score',
    'star_category',
    'log_budget',
    'log_boxoffice',
    'log_imdb_votes'
]
continuous_cols = [col for col in continuous_cols if col in X_train.columns]

scaler = StandardScaler()
X_train_scaled = X_train.copy()
X_test_scaled = X_test.copy()

# fit 只能在训练集执行；测试集只使用同一套规则 transform
X_train_scaled[continuous_cols] = scaler.fit_transform(X_train[continuous_cols])
X_test_scaled[continuous_cols] = scaler.transform(X_test[continuous_cols])

ridge_model = Ridge(alpha=1.0)
ridge_cv_scores = cross_val_score(
    ridge_model,
    X_train_scaled,
    y_train,
    cv=5,
    scoring='r2'
)

ridge_model.fit(X_train_scaled, y_train)
ridge_pred = ridge_model.predict(X_test_scaled)

ridge_r2 = r2_score(y_test, ridge_pred)
ridge_rmse = np.sqrt(mean_squared_error(y_test, ridge_pred))

print('Ridge average CV R2:', ridge_cv_scores.mean())
print('Ridge test R2:', ridge_r2)
print('Ridge test RMSE:', ridge_rmse)
```

## GridSearchCV 调优 Random Forest

`GridSearchCV` 不会创建新模型；它仍然使用 Random Forest，只是自动尝试不同的参数组合，并对每一组做 5-fold cross validation。

- `n_estimators`：森林中有多少棵树；更多树通常更稳定，但更慢。
- `max_depth`：每棵树能长多深；太深可能记住训练数据而过拟合。
- `min_samples_leaf`：叶节点至少要有多少样本；数值增大能降低过拟合风险。

```python
rf = RandomForestRegressor(
    random_state=42,
    n_jobs=-1
)

param_grid = {
    'n_estimators': [200, 300],
    'max_depth': [None, 12, 18],
    'min_samples_leaf': [1, 2, 5]
}

rf_search = GridSearchCV(
    estimator=rf,
    param_grid=param_grid,
    cv=5,
    scoring='r2',
    n_jobs=-1,
    refit=True
)

# Random Forest 不需要使用 X_train_scaled
rf_search.fit(X_train, y_train)

best_rf = rf_search.best_estimator_
rf_pred = best_rf.predict(X_test)

rf_r2 = r2_score(y_test, rf_pred)
rf_rmse = np.sqrt(mean_squared_error(y_test, rf_pred))
rf_mae = mean_absolute_error(y_test, rf_pred)

print('Best parameters:', rf_search.best_params_)
print('Best CV R2:', rf_search.best_score_)
print('Test MAE:', rf_mae)
print('Test RMSE:', rf_rmse)
print('Test R2:', rf_r2)
```

最终模型应主要根据 CV R2 是否稳定提升来选择；不要只因为某一次测试集 R2 更高，就认定模型更好。

## 模型比较与特征重要性

将同一测试集上的结果放在表里，便于向老师或业务方解释为什么选择 Random Forest。

```python
comparison = pd.DataFrame({
    'model': ['Ridge (scaled)', 'Random Forest (tuned)'],
    'cv_r2': [ridge_cv_scores.mean(), rf_search.best_score_],
    'test_r2': [ridge_r2, rf_r2],
    'test_rmse': [ridge_rmse, rf_rmse]
}).sort_values('test_r2', ascending=False)

print(comparison)

comparison.plot(
    x='model',
    y=['cv_r2', 'test_r2'],
    kind='bar',
    ylim=(0, 1),
    rot=0,
    title='Model Comparison: R2'
)
plt.ylabel('R2')
plt.show()
```

Random Forest 的 `feature_importances_` 能显示模型在哪些特征上最常做有效切分。它是业务探索的起点，不是因果结论。

```python
importance_df = pd.DataFrame({
    'feature': X_train.columns,
    'importance': best_rf.feature_importances_
}).sort_values('importance', ascending=False)

print(importance_df.head(15))

top_features = importance_df.head(15).sort_values('importance')

plt.figure(figsize=(9, 6))
plt.barh(top_features['feature'], top_features['importance'])
plt.title('Top 15 Random Forest Feature Importances')
plt.xlabel('Relative importance')
plt.show()
```

## BA 视角与注意事项

### 如何使用特征重要性

特征重要性回答的是“模型预测时最依赖什么”，不能直接回答“改变这个变量一定会造成观看量上升”。正确的使用顺序是：

1. 用重要性找到值得优先分析的变量。
2. 判断变量是否可控。
3. 对可控变量做实验或策略调整。
4. 用 A/B test 或准实验验证是否真的带来提升。

例如：

- 首页位置重要：可提出首页排序或曝光资源的 A/B test。
- `imdb_votes` 或 `boxoffice` 重要：更适合作为内容采购、选片和预估潜力的信号，不能直接被平台操控。
- `budget_missing` 重要：需要检查合作方资料质量、内容 metadata 覆盖度或长尾内容管理，而不是只把缺失值补掉。

### 不要机械照搬老师的所有处理

- 对 `budget` 等字段：本项目建议保留 missing indicator，再填补数值；这兼顾模型可训练性和 BA 的业务问题意识。
- 对 `release_year`：树模型保留原始年份即可；年份分箱主要是线性模型的可选增强。
- 对标准化：Ridge / Lasso、KNN、SVM、PCA 等需要；Random Forest、Gradient Boosting 等树模型通常不需要。
- 对数转换应先于标准化：先用 `log1p()` 压缩强右偏变量，再进行标准化。

## 复习问题

1. 为什么伪缺失的 0 值要同时保留 missing indicator 并进行填补？
2. 稀有 genre 为什么可能让模型过拟合？
3. 为什么 `StandardScaler` 必须只在训练集上 `fit`？
4. `GridSearchCV` 和 `RandomForestRegressor` 分别负责什么？
5. 为什么特征重要性不能直接当作因果结论？
