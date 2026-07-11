# Amazon Prime Video 观看量预测 - Colab Notebook 导出

> 这个文件是从原 Colab notebook `TVtext (2).ipynb` 导出的复习版，保留主要代码、关键输出和建模结论。原始 notebook 文件本地保存为 `amazon_prime_video_prediction.ipynb`。

## Notebook 结构

1. 读取数据与理解字段
2. 缺失值与 0 值检查
3. 特征工程与目标变量 log 转换
4. EDA：分布、相关性、位置、类型、metadata 缺失
5. 建模前处理：删除无用字段、genre dummy、类别变量 one-hot
6. 模型比较：Ridge、Random Forest、Gradient Boosting
7. 最终选择 Random Forest


## Cell 1: code

**这一步在做什么：** 读取 Amazon Prime Video 数据，确认数据规模为 4226 行、16 列。目标变量是 `cvt_per_day`。

```python
import pandas as pd

url = 'https://drive.google.com/uc?export=download&id=1vMUTPkfZOv-m6lQbIFVH65VpNnWMj7IN'
TV_df = pd.read_csv(url)

print(TV_df.shape)
TV_df.head()
```

**关键输出：**

```text
(4226, 16)


   video_id    cvt_per_day  weighted_categorical_position  \
0    385504  307127.605608                              1
1    300175  270338.426375                              1
2    361899  256165.867446                              1
3    308314  196622.720996                              3
4    307201  159841.652064                              1

   weighted_horizontal_poition  import_id  release_year  \
0                            3  lionsgate          2013
1                            3  lionsgate          2013
2                            3      other          2012
3                            4  lionsgate          2008
4                            3  lionsgate          2013

                                          genres  imdb_votes    budget  \
0                          Action,Thriller,Drama       69614  15000000
1                          Comedy,Crime,Thriller       46705  15000000
2                                    Crime,Drama      197596  26000000
3  Thriller,Drama,War,Documentary,Mystery,Action      356339  15000000
4             Crime,Thriller,Mystery,Documentary       46720  27220000

   boxoffice  imdb_rating  duration_in_mins  metacritic_score       awards  \
0   42930462          6.5        112.301017                51  other award
1    3301046          6.5         94.983250                41     no award
2   37397291          7.3        115.763675                58  other award
3   15700000          7.6        130.703583                94        Oscar
4    8551228          6.4        105.545533                37  other award

    mpaa  star_category
0  PG-13       1.710000
1      R       3.250000
2      R       2.646667
3      R       1.666667
4      R       3.066667
```


## Cell 2: code

**这一步在做什么：** 检查每列数据类型。注意 `import_id`、`genres`、`awards`、`mpaa` 是文字类型，建模前需要转成数值。

```python
TV_df.info()
```

**关键输出：**

```text
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4226 entries, 0 to 4225
Data columns (total 16 columns):
 #   Column                         Non-Null Count  Dtype
---  ------                         --------------  -----
 0   video_id                       4226 non-null   int64
 1   cvt_per_day                    4226 non-null   float64
 2   weighted_categorical_position  4226 non-null   int64
 3   weighted_horizontal_poition    4226 non-null   int64
 4   import_id                      4226 non-null   object
 5   release_year                   4226 non-null   int64
 6   genres                         4226 non-null   object
 7   imdb_votes                     4226 non-null   int64
 8   budget                         4226 non-null   int64
 9   boxoffice                      4226 non-null   int64
 10  imdb_rating                    4226 non-null   float64
 11  duration_in_mins               4226 non-null   float64
 12  metacritic_score               4226 non-null   int64
 13  awards                         4226 non-null   object
 14  mpaa                           4226 non-null   object
 15  star_category                  4226 non-null   float64
dtypes: float64(4), int64(8), object(4)
memory usage: 528.4+ KB
```


## Cell 3: code

**这一步在做什么：** 查看每列唯一值数量，判断哪些变量是连续型、离散型或高基数文本。

```python
TV_df.nunique()
```

**关键输出：**

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
dtype: int64
```


## Cell 4: code

**这一步在做什么：** 检查真正的 missing value。结果显示每列显式缺失值都是 0。

```python
missing_count=TV_df.isnull().sum()
print(missing_count)
```

**关键输出：**

```text
video_id                         0
cvt_per_day                      0
weighted_categorical_position    0
weighted_horizontal_poition      0
import_id                        0
release_year                     0
genres                           0
imdb_votes                       0
budget                           0
boxoffice                        0
imdb_rating                      0
duration_in_mins                 0
metacritic_score                 0
awards                           0
mpaa                             0
star_category                    0
dtype: int64
```


## Cell 5: code

**这一步在做什么：** 检查 0 值。这里的 0 很关键：budget、boxoffice、metacritic_score、star_category 中大量 0 更像“信息缺失”，不是实际为 0。

```python
zero_count=(TV_df==0).sum()
print(zero_count)
#No explicit missing values were found, but several numeric variables contain many zero values. For fields such as budget, boxoffice, metacritic_score, and star_category, zero may indicate unavailable information rather than a true value.
```

**关键输出：**

```text
video_id                            0
cvt_per_day                         0
weighted_categorical_position       0
weighted_horizontal_poition         0
import_id                           0
release_year                        0
genres                              0
imdb_votes                        344
budget                           2454
boxoffice                        3194
imdb_rating                       344
duration_in_mins                    0
metacritic_score                 3012
awards                              0
mpaa                                0
star_category                    1846
dtype: int64
```


## Cell 6: code

```python
print(TV_df.dtypes)
```

**关键输出：**

```text
video_id                           int64
cvt_per_day                      float64
weighted_categorical_position      int64
weighted_horizontal_poition        int64
import_id                         object
release_year                       int64
genres                            object
imdb_votes                         int64
budget                             int64
boxoffice                          int64
imdb_rating                      float64
duration_in_mins                 float64
metacritic_score                   int64
awards                            object
mpaa                              object
star_category                    float64
dtype: object
```


## Cell 7: code

```python
TV_df.describe()
```

**关键输出：**

```text
video_id    cvt_per_day  weighted_categorical_position  \
count    4226.000000    4226.000000                    4226.000000
mean   280371.162565    4218.630239                       7.782537
std    112640.127822   13036.079964                       6.134183
min      7909.000000       2.187625                       1.000000
25%    285104.250000     351.168776                       4.000000
50%    313891.500000    1193.499989                       6.000000
75%    349345.750000    3356.788816                       9.000000
max    394880.000000  307127.605608                      41.000000

       weighted_horizontal_poition  release_year     imdb_votes        budget  \
count                  4226.000000   4226.000000    4226.000000  4.226000e+03
mean                     28.103644   2001.056791    6462.924042  2.150743e+06
std                      11.863649     17.496849   31596.006790  7.176604e+06
min                       1.000000   1916.000000       0.000000  0.000000e+00
25%                      20.000000   1998.000000      81.000000  0.000000e+00
50%                      28.000000   2008.000000     535.000000  0.000000e+00
75%                      36.000000   2012.000000    3053.000000  1.500000e+06
max                      70.000000   2017.000000  948630.000000  1.070000e+08

          boxoffice  imdb_rating  duration_in_mins  metacritic_score  \
count  4.226000e+03  4226.000000       4226.000000       4226.000000
mean   2.536338e+06     5.257099         89.556123         15.973734
std    8.243516e+06     2.122810         21.086183         26.205217
min    0.000000e+00     0.000000          4.037250          0.000000
25%    0.000000e+00     4.300000         82.601712          0.000000
50%    0.000000e+00     5.800000         90.730308          0.000000
75%    0.000000e+00     6.800000         99.500312         41.000000
max    1.842088e+08    10.000000        246.016767        100.000000

       star_category
count    4226.000000
mean        0.954651
std         0.955045
min         0.000000
25%         0.000000
50%         1.000000
75%         1.666667
max         4.000000
```


## Cell 8: code

**这一步在做什么：** 创建 missing indicator，并对严重右偏的数值变量做 log transform，减少极端值影响。

```python
import numpy as np

# Create indicators for zero values that may represent missing information
TV_df['budget_missing'] = (TV_df['budget'] == 0).astype(int)
TV_df['boxoffice_missing'] = (TV_df['boxoffice'] == 0).astype(int)
TV_df['metacritic_missing'] = (TV_df['metacritic_score'] == 0).astype(int)
TV_df['star_category_missing'] = (TV_df['star_category'] == 0).astype(int)

# Log transform skewed numerical variables
TV_df['log_budget'] = np.log1p(TV_df['budget'])
TV_df['log_boxoffice'] = np.log1p(TV_df['boxoffice'])
TV_df['log_imdb_votes'] = np.log1p(TV_df['imdb_votes'])
```


## Cell 9: code

```python
TV_df.head()
```

**关键输出：**

```text
video_id    cvt_per_day  weighted_categorical_position  \
0    385504  307127.605608                              1
1    300175  270338.426375                              1
2    361899  256165.867446                              1
3    308314  196622.720996                              3
4    307201  159841.652064                              1

   weighted_horizontal_poition  import_id  release_year  \
0                            3  lionsgate          2013
1                            3  lionsgate          2013
2                            3      other          2012
3                            4  lionsgate          2008
4                            3  lionsgate          2013

                                          genres  imdb_votes    budget  \
0                          Action,Thriller,Drama       69614  15000000
1                          Comedy,Crime,Thriller       46705  15000000
2                                    Crime,Drama      197596  26000000
3  Thriller,Drama,War,Documentary,Mystery,Action      356339  15000000
4             Crime,Thriller,Mystery,Documentary       46720  27220000

   boxoffice  ...       awards   mpaa  star_category budget_missing  \
0   42930462  ...  other award  PG-13       1.710000              0
1    3301046  ...     no award      R       3.250000              0
2   37397291  ...  other award      R       2.646667              0
3   15700000  ...        Oscar      R       1.666667              0
4    8551228  ...  other award      R       3.066667              0

  boxoffice_missing  metacritic_missing  star_category_missing  log_budget  \
0                 0                   0                      0   16.523561
1                 0                   0                      0   16.523561
2                 0                   0                      0   17.073607
3                 0                   0                      0   16.523561
4                 0                   0                      0   17.119463

   log_boxoffice  log_imdb_votes
0      17.575092       11.150735
1      15.009750       10.751628
2      17.437109       12.193985
3      16.569171       12.783641
4      15.961586       10.751949

[5 rows x 23 columns]
```


## Cell 10: code

```python
import matplotlib.pyplot as plt
import seaborn as sns

sns.histplot(TV_df['cvt_per_day'], bins=50)
plt.title('Distribution of cvt_per_day')
plt.xlabel('cvt_per_day')
plt.ylabel('Number of movies')
plt.show()
```

**关键输出：**

```text
<Figure size 640x480 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 11: code

**这一步在做什么：** 目标变量 `cvt_per_day` 严重右偏，所以使用 `log_cvt_per_day` 作为建模目标。

```python
import numpy as np

TV_df['log_cvt_per_day'] = np.log1p(TV_df['cvt_per_day'])

sns.histplot(TV_df['log_cvt_per_day'], bins=50)
plt.title('Distribution of log(cvt_per_day)')
plt.xlabel('log(cvt_per_day)')
plt.ylabel('Number of movies')
plt.show()
#原始 cvt_per_day 严重右偏，大多数电影观看量较低，少数电影观看量极高。经过 log 转换后，分布变得更平衡、更接近正态分布，因此后续建模可以使用 log_cvt_per_day 作为目标变量。
```

**关键输出：**

```text
<Figure size 640x480 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 12: code

**这一步在做什么：** 看数值变量与目标变量的相关性：IMDb votes、boxoffice、star_category、budget、duration、metacritic 与观看量正相关；位置变量与观看量负相关。

```python
num_cols = TV_df.select_dtypes(include=['int64', 'float64']).columns

corr_with_target = TV_df[num_cols].corr()['log_cvt_per_day'].sort_values(ascending=False)
corr_with_target
```

**关键输出：**

```text
log_cvt_per_day                  1.000000
cvt_per_day                      0.535691
log_imdb_votes                   0.484882
log_boxoffice                    0.424060
star_category                    0.408017
log_budget                       0.384185
duration_in_mins                 0.352829
metacritic_score                 0.331132
video_id                         0.273980
budget                           0.269661
boxoffice                        0.264159
imdb_votes                       0.175934
imdb_rating                      0.102501
release_year                     0.002840
weighted_categorical_position   -0.229403
weighted_horizontal_poition     -0.240495
budget_missing                  -0.342196
metacritic_missing              -0.366425
boxoffice_missing               -0.420517
star_category_missing           -0.432301
Name: log_cvt_per_day, dtype: float64
```


## Cell 13: code

```python
plt.figure(figsize=(12, 8))
sns.heatmap(TV_df[num_cols].corr(), cmap='coolwarm', center=0)
plt.title('Correlation Heatmap of Numerical Variables')
plt.show()
```

**关键输出：**

```text
<Figure size 1200x800 with 2 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 14: code

```python
sns.scatterplot(data=TV_df, x='log_imdb_votes', y='log_cvt_per_day', alpha=0.4)
plt.title('log_imdb_votes vs log_cvt_per_day')
plt.show()
#IMDb 投票数越高的电影，平台每日观看量通常也越高，说明外部热度和平台消费表现有正相关。但点分布仍然比较分散，说明 IMDb votes 不能单独解释观看量，还需要结合曝光位置、类型、票房、明星热度等变量。
```

**关键输出：**

```text
<Figure size 640x480 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 15: code

**这一步在做什么：** 修正原始字段拼写错误：`weighted_horizontal_poition` 改为 `weighted_horizontal_position`。

```python
TV_df = TV_df.rename(columns={
    'weighted_horizontal_poition': 'weighted_horizontal_position'
})
TV_df.head()
```

**关键输出：**

```text
video_id    cvt_per_day  weighted_categorical_position  \
0    385504  307127.605608                              1
1    300175  270338.426375                              1
2    361899  256165.867446                              1
3    308314  196622.720996                              3
4    307201  159841.652064                              1

   weighted_horizontal_position  import_id  release_year  \
0                             3  lionsgate          2013
1                             3  lionsgate          2013
2                             3      other          2012
3                             4  lionsgate          2008
4                             3  lionsgate          2013

                                          genres  imdb_votes    budget  \
0                          Action,Thriller,Drama       69614  15000000
1                          Comedy,Crime,Thriller       46705  15000000
2                                    Crime,Drama      197596  26000000
3  Thriller,Drama,War,Documentary,Mystery,Action      356339  15000000
4             Crime,Thriller,Mystery,Documentary       46720  27220000

   boxoffice  ...   mpaa  star_category  budget_missing boxoffice_missing  \
0   42930462  ...  PG-13       1.710000               0                 0
1    3301046  ...      R       3.250000               0                 0
2   37397291  ...      R       2.646667               0                 0
3   15700000  ...      R       1.666667               0                 0
4    8551228  ...      R       3.066667               0                 0

  metacritic_missing  star_category_missing  log_budget  log_boxoffice  \
0                  0                      0   16.523561      17.575092
1                  0                      0   16.523561      15.009750
2                  0                      0   17.073607      17.437109
3                  0                      0   16.523561      16.569171
4                  0                      0   17.119463      15.961586

   log_imdb_votes  log_cvt_per_day
0       11.150735        12.635022
1       10.751628        12.507434
2       12.193985        12.453584
3       12.783641        12.189047
4       10.751949        11.981945

[5 rows x 24 columns]
```


## Cell 16: code

**这一步在做什么：** 可视化 horizontal position 与观看量的关系：位置数字越大，通常越靠后，观看量越低。

```python
sns.scatterplot(data=TV_df, x='weighted_horizontal_position', y='log_cvt_per_day', alpha=0.4)
plt.title('weighted_horizontal_position vs log_cvt_per_day')
plt.xlabel('weighted_horizontal_position')
plt.ylabel('log_cvt_per_day')
plt.show()
#电影在页面上越靠前/越靠左，通常获得更多曝光，因此每日观看量更高；位置越靠后，观看量通常更低。
```

**关键输出：**

```text
<Figure size 640x480 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 17: code

**这一步在做什么：** 用 boxplot 看 MPAA 分级与观看量差异。

```python
sns.boxplot(data=TV_df, x='mpaa', y='log_cvt_per_day')
plt.title('MPAA rating vs log_cvt_per_day')
plt.xlabel('MPAA rating')
plt.ylabel('log_cvt_per_day')
plt.show()
#电影分级和观看表现可能有一定关系。R、PG、NC-17 等分级的电影中位观看量相对更高，而 NotRated 偏低。但每个分级内部差异很大，说明 MPAA 分级不能单独解释电影受欢迎程度。
```

**关键输出：**

```text
<Figure size 640x480 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 18: code

**这一步在做什么：** 按内容供应商 `import_id` 汇总观看表现，Paramount/Lionsgate/MGM 明显高于 other。

```python
TV_df.groupby('import_id')['log_cvt_per_day'].agg(['count', 'mean', 'median']).sort_values('median', ascending=False)
```

**关键输出：**

```text
count      mean    median
import_id
paramount    141  8.580583  8.727663
lionsgate    677  8.357836  8.280992
mgm          445  8.039370  8.194930
other       2963  6.508061  6.563356
```


## Cell 19: code

**这一步在做什么：** 把多标签 `genres` 拆成 genre dummy，比较不同类型电影的观看表现。

```python
#这一步主要是看一下，单一类型对于观看次数的影响
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

**关键输出：**

```text
genre  count  mean_log_cvt  median_log_cvt
11                Holiday      1      8.338200        8.338200
3               Animation    129      7.644453        7.723621
14          Kids & Family    280      7.582852        7.719710
6                   Crime    437      7.615153        7.637322
2               Adventure    363      7.510770        7.599680
24               Thriller    879      7.481472        7.561070
9                 Fantasy    243      7.506231        7.555721
0                  Action    739      7.534424        7.538744
15                   LGBT      2      7.538456        7.538456
26                Western    102      7.457579        7.537023
22                 Sci-Fi    363      7.328994        7.404624
19                Mystery    375      7.274344        7.354253
21                Romance    591      7.233153        7.294094
12                 Horror    762      7.289995        7.249296
8                   Drama   1677      7.178139        7.200978
4                   Anime     11      7.432012        7.181491
5                  Comedy   1184      7.064650        7.138202
13            Independent    393      6.894901        7.000090
25                    War    102      7.108004        6.959918
10  Foreign/International     64      6.694978        6.928276
23                  Sport     77      6.676120        6.672747
20                Reality      9      6.653202        6.628107
18               Musicals     68      6.522061        6.620077
1                   Adult      3      6.510085        6.425037
7             Documentary    671      6.209174        6.241210
17                  Music    171      5.930626        5.936729
16              Lifestyle      7      6.137829        5.526353
```


## Cell 20: code

**这一步在做什么：** 按 IMDb 热度分组后再看位置影响：即使电影本身热度高，位置靠后也会拉低观看量。

```python
#这一步主要看一下即使自身热度非常高的电影，是否也会受到位置的影响
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
#按 IMDb 热度分组后，高热度电影整体观看量更高。但在中高热度组内部，随着页面位置数字变大，观看量有下降趋势。这说明即使考虑电影本身热度，页面展示位置仍然会影响观看表现。
```

**关键输出：**

```text
<Figure size 880.75x500 with 1 Axes>
```

**图表输出：** 该 cell 在 Colab 中生成了 1 张图；GitHub 复习版保留代码和图表解读。


## Cell 21: code

**这一步在做什么：** 验证 metadata 缺失是否影响表现：budget/boxoffice/metacritic/star 信息缺失的电影观看量明显更低。

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

**关键输出：**

```text
budget_missing
                count      mean    median
budget_missing
0                1772  7.677264  7.737752
1                2454  6.570868  6.624512

 boxoffice_missing
                   count      mean    median
boxoffice_missing
0                   1032  8.215074  8.343983
1                   3194  6.653433  6.724232

 metacritic_missing
                    count      mean    median
metacritic_missing
0                    1214  7.955619  8.040132
1                    3012  6.663646  6.733419

 star_category_missing
                       count      mean    median
star_category_missing
0                       2380  7.642209  7.712335
1                       1846  6.251660  6.281770
```


## Cell 22: code

**这一步在做什么：** 建模前删除无预测意义或会造成泄漏的列：`video_id`、原始目标变量、原始 genres、原始高度偏斜变量等。

```python
#数据的预出楼阶段
#Step1:删除所有对于训练模型没有意义的数据
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

X.head()
```

**关键输出：**

```text
weighted_categorical_position  weighted_horizontal_position  import_id  \
0                              1                             3  lionsgate
1                              1                             3  lionsgate
2                              1                             3      other
3                              3                             4  lionsgate
4                              1                             3  lionsgate

   release_year  imdb_rating  duration_in_mins  metacritic_score       awards  \
0          2013          6.5        112.301017                51  other award
1          2013          6.5         94.983250                41     no award
2          2012          7.3        115.763675                58  other award
3          2008          7.6        130.703583                94        Oscar
4          2013          6.4        105.545533                37  other award

    mpaa  star_category  budget_missing  boxoffice_missing  \
0  PG-13       1.710000               0                  0
1      R       3.250000               0                  0
2      R       2.646667               0                  0
3      R       1.666667               0                  0
4      R       3.066667               0                  0

   metacritic_missing  star_category_missing  log_budget  log_boxoffice  \
0                   0                      0   16.523561      17.575092
1                   0                      0   16.523561      15.009750
2                   0                      0   17.073607      17.437109
3                   0                      0   16.523561      16.569171
4                   0                      0   17.119463      15.961586

   log_imdb_votes popularity_group
0       11.150735  High popularity
1       10.751628  High popularity
2       12.193985  High popularity
3       12.783641  High popularity
4       10.751949  High popularity
```


## Cell 23: code

**这一步在做什么：** 把 `genres` 拆成多个 0/1 类型变量，供模型使用。

```python
genre_dummies = TV_df['genres'].str.get_dummies(sep=',')
genre_dummies = genre_dummies.add_prefix('genre_')
X = pd.concat([X, genre_dummies], axis=1)
X = X.drop(columns=['popularity_group'])
X.head()
```

**关键输出：**

```text
weighted_categorical_position  weighted_horizontal_position  release_year  \
0                              1                             3          2013
1                              1                             3          2013
2                              1                             3          2012
3                              3                             4          2008
4                              1                             3          2013

   imdb_rating  duration_in_mins  metacritic_score  star_category  \
0          6.5        112.301017                51       1.710000
1          6.5         94.983250                41       3.250000
2          7.3        115.763675                58       2.646667
3          7.6        130.703583                94       1.666667
4          6.4        105.545533                37       3.066667

   budget_missing  boxoffice_missing  metacritic_missing  ...  genre_Music  \
0               0                  0                   0  ...            0
1               0                  0                   0  ...            0
2               0                  0                   0  ...            0
3               0                  0                   0  ...            0
4               0                  0                   0  ...            0

   genre_Musicals  genre_Mystery  genre_Reality  genre_Romance  genre_Sci-Fi  \
0               0              0              0              0             0
1               0              0              0              0             0
2               0              0              0              0             0
3               0              1              0              0             0
4               0              1              0              0             0

   genre_Sport  genre_Thriller  genre_War  genre_Western
0            0               1          0              0
1            0               1          0              0
2            0               0          0              0
3            0               1          1              0
4            0               1          0              0

[5 rows x 83 columns]
```


## Cell 24: code

```python
X.select_dtypes(include='object').columns
```

**关键输出：**

```text
Index(['import_id', 'awards', 'mpaa'], dtype='object')
```


## Cell 25: code

**这一步在做什么：** 把剩余文字变量 `import_id`、`awards`、`mpaa` one-hot encoding，确保模型只能接收数字。

```python
X = pd.get_dummies(X, columns=['import_id', 'awards', 'mpaa'])
X.select_dtypes(include='object').columns
```

**关键输出：**

```text
Index([], dtype='object')
```


## Cell 26: code

**这一步在做什么：** 训练 Ridge regression 作为线性基准模型，并用 5-fold cross validation 评估。

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=42
)
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

**关键输出：**

```text
[0.40793143 0.43959005 0.40596823 0.49933885 0.45493015]
0.44155174359719573
```


## Cell 27: code

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

mae = mean_absolute_error(y_test, y_pred)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
r2 = r2_score(y_test, y_pred)

print('Test MAE:', mae)
print('Test RMSE:', rmse)
print('Test R2:', r2)
```

**关键输出：**

```text
Test MAE: 0.8883665958787954
Test RMSE: 1.1619833560034754
Test R2: 0.48232061220500855
```


## Cell 28: code

**这一步在做什么：** 训练 Random Forest。它能捕捉非线性关系，测试集表现最好。

```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import cross_val_score
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np
#我最终选择 Random Forest，因为它在测试集上的表现最好。同时它可以捕捉非线性关系，适合这个数据集里位置、类型、外部热度和 metadata 信息之间的复杂影响。

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

**关键输出：**

```text
[0.51454723 0.55911906 0.531854   0.57245515 0.55543946]
Average CV R2: 0.5466829803540577
Test MAE: 0.7741244894600299
Test RMSE: 1.0241083863359493
Test R2: 0.597882565030115
```


## Cell 29: code

**这一步在做什么：** 训练 Gradient Boosting 作对比，表现接近但略低于 Random Forest。

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

**关键输出：**

```text
[0.50923925 0.50034921 0.48797537 0.5618802  0.55616451]
Average CV R2: 0.5231217065359005
Test MAE: 0.8064301808317977
Test RMSE: 1.053969474287878
Test R2: 0.5740906992762881
```


## 最终模型结论

- 最终选择：Random Forest Regressor。
- 目标变量：`log_cvt_per_day`。
- 训练/测试划分：75% train，25% test，`random_state=42`。
- 验证方式：5-fold cross validation。
- Random Forest 表现：Average CV R2 = 0.5467，Test R2 = 0.5979，Test MAE = 0.7741，Test RMSE = 1.0241。
- 对比：Ridge Test R2 = 0.4823；Gradient Boosting Test R2 = 0.5741。


## 业务解释

这个模型不是为了证明某一个变量单独决定观看量，而是综合内容自身热度、页面位置、类型、供应商、奖项/分级、metadata 完整度等信息，预测电影在平台上的每日观看量。EDA 和模型结果共同说明：外部热度、页面曝光位置、内容 metadata 完整度是影响 `cvt_per_day` 的重要因素。
