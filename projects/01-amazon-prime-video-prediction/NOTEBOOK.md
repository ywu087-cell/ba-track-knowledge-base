# Amazon Prime Video 观看量预测 - Colab Notebook 复习版

> 这是从本次 Colab 项目整理出来的详细复习版。它保留主要代码、关键输出、每一步在做什么，以及最后为什么选择 Random Forest。适合以后不用回 Colab，也能复盘完整项目逻辑。

## 1. 读取数据

```python
import pandas as pd

url = 'https://drive.google.com/uc?export=download&id=1vMUTPkfZOv-m6lQbIFVH65VpNnWMj7IN'
TV_df = pd.read_csv(url)

print(TV_df.shape)
TV_df.head()
```

关键输出：

```text
(4226, 16)
```

含义：数据有 4226 行、16 列。每一行是一部电影，目标是预测 `cvt_per_day`。

## 2. 理解字段和数据类型

```python
TV_df.info()
```

关键输出：

```text
video_id                       int64
cvt_per_day                    float64
weighted_categorical_position  int64
weighted_horizontal_poition    int64
import_id                      object
release_year                   int64
genres                         object
imdb_votes                     int64
budget                         int64
boxoffice                      int64
imdb_rating                    float64
duration_in_mins               float64
metacritic_score               int64
awards                         object
mpaa                           object
star_category                  float64
```

重点：`import_id`、`genres`、`awards`、`mpaa` 是文字类型，模型不能直接识别，后面要转成数字。

## 3. 查看唯一值数量

```python
TV_df.nunique()
```

关键输出：

```text
video_id                         4226
cvt_per_day                      4226
weighted_categorical_position      37
weighted_horizontal_poition        68
import_id                           4
release_year                       97
genres                           1165
imdb_votes                       2282
budget                            253
boxoffice                         410
imdb_rating                        85
duration_in_mins                 4097
metacritic_score                   90
awards                              5
mpaa                                6
star_category                     630
```

理解：

- `video_id` 每行唯一，对预测没有实际意义。
- `genres` 有 1165 种组合，不能直接 one-hot 整个组合，应该拆成单个 genre dummy。
- `import_id`、`awards`、`mpaa` 类别数量不多，适合 one-hot encoding。

## 4. 检查显式缺失值

```python
missing_count = TV_df.isnull().sum()
print(missing_count)
```

结果：所有列都是 0。

含义：数据里没有 `NaN` 这种显式 missing value。

## 5. 检查 0 值

```python
zero_count = (TV_df == 0).sum()
print(zero_count)
```

关键输出：

```text
imdb_votes             344
budget                2454
boxoffice             3194
imdb_rating            344
metacritic_score      3012
star_category         1846
```

重点：这里的 0 很可能不是“真实为 0”，而是“信息缺失”。比如电影预算、票房、Metacritic score 不太可能真的都是 0，所以后面要做 missing indicator。

## 6. 创建 missing indicator 和 log 特征

```python
import numpy as np

TV_df['budget_missing'] = (TV_df['budget'] == 0).astype(int)
TV_df['boxoffice_missing'] = (TV_df['boxoffice'] == 0).astype(int)
TV_df['metacritic_missing'] = (TV_df['metacritic_score'] == 0).astype(int)
TV_df['star_category_missing'] = (TV_df['star_category'] == 0).astype(int)

TV_df['log_budget'] = np.log1p(TV_df['budget'])
TV_df['log_boxoffice'] = np.log1p(TV_df['boxoffice'])
TV_df['log_imdb_votes'] = np.log1p(TV_df['imdb_votes'])
```

为什么这么做：

- missing indicator 保留“这个信息是否缺失”的信号。
- log transform 用来处理严重右偏的数值，比如票房、预算、IMDb votes。
- `np.log1p(x)` 等于 `log(1+x)`，可以处理 0 值。

## 7. 目标变量 log 转换

```python
TV_df['log_cvt_per_day'] = np.log1p(TV_df['cvt_per_day'])

sns.histplot(TV_df['log_cvt_per_day'], bins=50)
plt.title('Distribution of log(cvt_per_day)')
plt.xlabel('log(cvt_per_day)')
plt.ylabel('Number of movies')
plt.show()
```

解释：原始 `cvt_per_day` 极度右偏，大多数电影观看量低，少数电影观看量极高。做 log 后分布更稳定，所以后面用 `log_cvt_per_day` 做目标变量。

## 8. 数值相关性分析

```python
num_cols = TV_df.select_dtypes(include=['int64', 'float64']).columns
corr_with_target = TV_df[num_cols].corr()['log_cvt_per_day'].sort_values(ascending=False)
corr_with_target
```

关键输出：

```text
log_imdb_votes                   0.484882
log_boxoffice                    0.424060
star_category                    0.408017
log_budget                       0.384185
duration_in_mins                 0.352829
metacritic_score                 0.331132
weighted_categorical_position   -0.229403
weighted_horizontal_poition     -0.240495
budget_missing                  -0.342196
metacritic_missing              -0.366425
boxoffice_missing               -0.420517
star_category_missing           -0.432301
```

业务解释：

- 外部热度越高，比如 IMDb votes、boxoffice，平台观看量通常越高。
- 页面位置数字越大，通常位置越靠后，观看量越低。
- metadata 缺失的电影表现更差，这可能反映电影本身知名度较低、信息不完整或内容质量/曝光较弱。

## 9. 修正字段名

```python
TV_df = TV_df.rename(columns={
    'weighted_horizontal_poition': 'weighted_horizontal_position'
})
```

原始数据里 `position` 拼错成了 `poition`，这里统一改正，避免后续代码反复报错。

## 10. 位置和观看量关系

```python
sns.scatterplot(
    data=TV_df,
    x='weighted_horizontal_position',
    y='log_cvt_per_day',
    alpha=0.4
)
plt.title('weighted_horizontal_position vs log_cvt_per_day')
plt.show()
```

结论：位置数字越大，观看量通常越低。也就是说，即使电影本身有热度，首页展示位置仍然会影响消费表现。

## 11. MPAA 分级分析

```python
sns.boxplot(data=TV_df, x='mpaa', y='log_cvt_per_day')
plt.title('MPAA rating vs log_cvt_per_day')
plt.show()
```

结论：不同 MPAA 分级的观看量分布不同，但每个分级内部差异也很大，所以 MPAA 不能单独解释观看量，只能作为辅助特征。

## 12. 内容供应商表现

```python
TV_df.groupby('import_id')['log_cvt_per_day'].agg(['count', 'mean', 'median']).sort_values('median', ascending=False)
```

关键输出：

```text
           count      mean    median
paramount    141  8.580583  8.727663
lionsgate    677  8.357836  8.280992
mgm          445  8.039370  8.194930
other       2963  6.508061  6.563356
```

结论：Paramount、Lionsgate、MGM 的中位观看量明显高于 other，说明内容供应商本身可能代表内容质量、版权价值或用户偏好。

## 13. Genre dummy 和类型分析

```python
genre_dummies = TV_df['genres'].str.get_dummies(sep=',')
genre_summary = []

for genre in genre_dummies.columns:
    temp = TV_df[genre_dummies[genre] == 1]
    genre_summary.append({
        'genre': genre,
        'count': len(temp),
        'mean_log_cvt': temp['log_cvt_per_day'].mean(),
        'median_log_cvt': temp['log_cvt_per_day'].median()
    })

genre_summary = pd.DataFrame(genre_summary)
genre_summary.sort_values('median_log_cvt', ascending=False)
```

高观看量类型包括 Animation、Kids & Family、Crime、Adventure、Thriller、Fantasy、Action 等。

注意：有些类型样本很少，比如 Holiday、LGBT，所以不能只看 median，还要看 count。

## 14. 高热度电影是否也受位置影响

```python
TV_df['popularity_group'] = pd.qcut(
    TV_df['log_imdb_votes'],
    q=3,
    labels=['Low popularity', 'Medium popularity', 'High popularity']
)

sns.lmplot(
    data=TV_df,
    x='weighted_horizontal_position',
    y='log_cvt_per_day',
    hue='popularity_group',
    scatter_kws={'alpha': 0.3},
    height=5,
    aspect=1.4
)
plt.title('Position Effect by Popularity Group')
plt.show()
```

结论：高 IMDb 热度电影整体观看量更高，但在中高热度组里，位置靠后仍然会降低观看表现。这个 insight 比普通相关性更有价值，因为它说明位置不是单纯被“电影本身热度”解释掉的。

## 15. Metadata 缺失与观看表现

```python
missing_cols = [
    'budget_missing',
    'boxoffice_missing',
    'metacritic_missing',
    'star_category_missing'
]

for col in missing_cols:
    print('\n', col)
    print(TV_df.groupby(col)['log_cvt_per_day'].agg(['count', 'mean', 'median']))
```

关键输出：

```text
budget_missing = 0: median 7.737752
budget_missing = 1: median 6.624512

boxoffice_missing = 0: median 8.343983
boxoffice_missing = 1: median 6.724232

metacritic_missing = 0: median 8.040132
metacritic_missing = 1: median 6.733419

star_category_missing = 0: median 7.712335
star_category_missing = 1: median 6.281770
```

结论：metadata 完整的电影观看表现更好。对 BA 来说，这可以解释为：完整的外部信息往往代表电影知名度更高、商业化程度更高，或平台更容易识别和推荐。

## 16. 建模前处理：定义 y 和 X

```python
y = TV_df['log_cvt_per_day']

drop_cols = [
    'video_id',
    'cvt_per_day',
    'log_cvt_per_day',
    'genres',
    'budget',
    'boxoffice',
    'imdb_votes'
]

X = TV_df.drop(columns=drop_cols)
```

为什么 `X = drop(...)`：

- `X` 是模型用来预测的特征表。
- `drop_cols` 是不应该进入模型的列。
- 所以 `TV_df.drop(columns=drop_cols)` 的意思是：从原表里删除这些列，剩下的才是模型特征。

删除原因：

- `video_id`：只是编号，没有预测意义。
- `cvt_per_day`、`log_cvt_per_day`：目标变量，不能放进 X，否则数据泄漏。
- `genres`：原始多标签文本，模型不能直接用，后面用 dummy 替代。
- `budget`、`boxoffice`、`imdb_votes`：保留 log 版本，删除原始高度偏斜版本。

## 17. 加入 genre dummy

```python
genre_dummies = TV_df['genres'].str.get_dummies(sep=',')
genre_dummies = genre_dummies.add_prefix('genre_')
X = pd.concat([X, genre_dummies], axis=1)
X = X.drop(columns=['popularity_group'])
```

解释：

- 原始 `genres` 可能是 `Action,Thriller,Drama`。
- `str.get_dummies(sep=',')` 会拆成 `Action=1`、`Thriller=1`、`Drama=1`。
- 这样模型才能理解电影属于哪些类型。

## 18. 处理剩余文字变量

```python
X.select_dtypes(include='object').columns
```

关键输出：

```text
Index(['import_id', 'awards', 'mpaa'], dtype='object')
```

然后做 one-hot encoding：

```python
X = pd.get_dummies(X, columns=['import_id', 'awards', 'mpaa'])
X.select_dtypes(include='object').columns
```

关键输出：

```text
Index([], dtype='object')
```

含义：现在 X 里已经没有文字列，模型可以训练。

## 19. 划分训练集和测试集

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=42
)
```

解释：

- 75% 数据用于训练。
- 25% 数据用于最终测试。
- `random_state=42` 保证每次划分结果一致。

K-fold cross validation 仍然需要，但它只在训练集上做，不替代最终测试集。

## 20. Ridge 模型 baseline

```python
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import Ridge

ridge_model = Ridge(alpha=1.0)

cv_scores = cross_val_score(
    ridge_model,
    X_train,
    y_train,
    cv=5,
    scoring='r2'
)

print(cv_scores)
print(cv_scores.mean())

ridge_model.fit(X_train, y_train)
y_pred = ridge_model.predict(X_test)
```

结果：

```text
CV R2: [0.4079, 0.4396, 0.4060, 0.4993, 0.4549]
Average CV R2: 0.4416
Test R2: 0.4823
Test MAE: 0.8884
Test RMSE: 1.1620
```

解释：Ridge 是线性模型，适合作 baseline，但对复杂非线性关系捕捉不够。

## 21. Random Forest 模型

```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import cross_val_score
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

rf_model = RandomForestRegressor(
    n_estimators=300,
    random_state=42
)

rf_cv_scores = cross_val_score(
    rf_model,
    X_train,
    y_train,
    cv=5,
    scoring='r2'
)

print(rf_cv_scores)
print('Average CV R2:', rf_cv_scores.mean())

rf_model.fit(X_train, y_train)
rf_pred = rf_model.predict(X_test)

print('Test MAE:', mean_absolute_error(y_test, rf_pred))
print('Test RMSE:', np.sqrt(mean_squared_error(y_test, rf_pred)))
print('Test R2:', r2_score(y_test, rf_pred))
```

结果：

```text
CV R2: [0.5145, 0.5591, 0.5319, 0.5725, 0.5554]
Average CV R2: 0.5467
Test MAE: 0.7741
Test RMSE: 1.0241
Test R2: 0.5979
```

解释：Random Forest 表现最好，因为它能捕捉非线性关系和变量之间的复杂互动，例如：位置、外部热度、类型、metadata 缺失之间并不是简单线性关系。

## 22. Gradient Boosting 对比

```python
from sklearn.ensemble import GradientBoostingRegressor

gb_model = GradientBoostingRegressor(
    n_estimators=300,
    learning_rate=0.05,
    max_depth=3,
    random_state=42
)

gb_cv_scores = cross_val_score(
    gb_model,
    X_train,
    y_train,
    cv=5,
    scoring='r2'
)

print(gb_cv_scores)
print('Average CV R2:', gb_cv_scores.mean())

gb_model.fit(X_train, y_train)
gb_pred = gb_model.predict(X_test)

print('Test MAE:', mean_absolute_error(y_test, gb_pred))
print('Test RMSE:', np.sqrt(mean_squared_error(y_test, gb_pred)))
print('Test R2:', r2_score(y_test, gb_pred))
```

结果：

```text
CV R2: [0.5092, 0.5003, 0.4880, 0.5619, 0.5562]
Average CV R2: 0.5231
Test MAE: 0.8064
Test RMSE: 1.0540
Test R2: 0.5741
```

解释：Gradient Boosting 也不错，但这次测试集表现低于 Random Forest。

## 23. 模型选择结论

最终选择 Random Forest：

| 模型 | Average CV R2 | Test R2 | Test MAE | Test RMSE |
| --- | ---: | ---: | ---: | ---: |
| Ridge | 0.4416 | 0.4823 | 0.8884 | 1.1620 |
| Random Forest | 0.5467 | 0.5979 | 0.7741 | 1.0241 |
| Gradient Boosting | 0.5231 | 0.5741 | 0.8064 | 1.0540 |

Random Forest 在 cross validation 和 test set 上整体最好，所以作为最终模型。

## 24. BA 视角总结

这个项目的核心不是“模型越复杂越好”，而是：

- 先理解业务目标：预测电影每日观看量。
- 再识别数据问题：没有 NaN，但有大量 0 值代表伪缺失。
- 再做合理特征工程：log transform、missing indicator、genre dummy、one-hot encoding。
- 再比较模型：Ridge 作为 baseline，Random Forest 和 Gradient Boosting 捕捉非线性。
- 最后输出业务 insight：外部热度、页面位置、内容供应商、metadata 完整度都和观看表现相关。

最终可讲的 insight：

1. IMDb votes、boxoffice、star_category 越高，平台观看量通常越高。
2. 页面展示位置越靠后，观看量通常越低。
3. 即使高热度电影，也仍然受到页面位置影响。
4. budget/boxoffice/metacritic/star 信息缺失的电影，观看表现明显更低。
5. Random Forest 能较好综合这些非线性信号，测试集 R2 约 0.598。
