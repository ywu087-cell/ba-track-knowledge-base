# 第 02 课 - Data Driven Marketing、SEM 指标与客户旅程

日期：2026-07-06  
来源：BA track 第二节课 PDF，主题是 Data Driven Marketing 和 Search Engine Marketing 后续内容  
整理方式：中文解释为主，保留关键英文术语；不是逐页照搬 PDF

## 一句话总结

这节课把第一节的搜索引擎营销继续往业务分析方向推进：先用 Amazon Prime Video 数据集引出预测建模问题，再讲 SEM 的真实点击成本、效果指标、A/B test，最后连接到客户旅程和数字营销渠道。

## 快速目录

1. Data Challenge：Amazon Prime Video 数据集
2. 如何构建 `cvt_per_day` 预测模型
3. Actual CPC：真实点击成本
4. SEM 效果衡量指标
5. A/B Testing 与两比例 Z 检验
6. SEM 后台界面怎么读
7. SEM vs SEO
8. Customer Journey 与 Moment of Truth
9. Digital Marketing 与渠道
10. BA 视角总结

## 核心概念索引

- 数据驱动营销 (Data Driven Marketing)：用数据理解用户、评估渠道、优化投放和转化。
- 累计每日观看时长 (Cumulated View Time Per Day, `cvt_per_day`)：衡量视频内容受欢迎程度的目标变量。
- 特征工程 (Feature Engineering)：把原始字段转成更适合建模的解释变量。
- 缺失值处理 (Missing Value Handling)：在建模前处理空值，避免模型偏差或无法训练。
- 真实点击成本 (Actual Cost Per Click, Actual CPC)：广告主实际为一次点击支付的金额。
- 点击率 (Click Through Rate, CTR)：`Clicks / Impressions`。
- 转化率 (Conversion Rate, CVR)：`Conversions / Clicks`。
- 单次获客成本 (Cost Per Acquisition, CPA)：`Total Cost / Conversions`。
- A/B Testing：比较两个版本表现差异的实验方法。
- 两比例 Z 检验 (Two-proportion Z-test)：检验两个比例指标是否存在统计显著差异。
- 客户旅程 (Customer Journey)：用户从需求触发到购买、使用和分享的全过程。
- 关键真实时刻 (Moment of Truth, MOT)：用户与品牌、产品或服务接触并形成印象的关键瞬间。
- 数字营销 (Digital Marketing)：通过互联网、移动设备、展示广告等数字媒介进行营销。

## 1. Data Challenge：Amazon Prime Video 数据集

课程用 Amazon Prime Video 数据集作为数据挑战。数据大约有 4000 多行、16 个字段，核心目标是预测 `cvt_per_day`，也就是某个视频内容每天被观看的累计时长。

可以把这个问题理解成：

```text
内容属性 + 曝光位置 + 外部评价 + 制作信息 -> 预测内容每天被观看多少
```

数据字段大致分为几类：

| 类别 | 字段例子 | BA 解读 |
| --- | --- | --- |
| 目标变量 | `cvt_per_day` | 要预测的内容消费表现 |
| 平台曝光 | `weighted_categorical_position`, `weighted_horizontal_position` | 内容在首页或分类页的位置，越靠前通常越容易被看到 |
| 内容属性 | `Genres`, `release_year`, `duration_in_mins`, `mpaa` | 内容类型、年份、时长、分级 |
| 外部口碑 | `imdb_votes`, `imdb_rating`, `Metacritic_score`, `Awards` | 用户评价、评分、奖项或媒体评价 |
| 商业属性 | `budget`, `boxoffice`, `Import_id`, `Star_category` | 制作投入、票房、合作方、演员知名度 |

这类数据挑战的核心不是“套一个模型就结束”，而是要说明：哪些变量可能影响观看表现，数据质量如何，模型结果是否能转成业务建议。

## 2. 如何构建 `cvt_per_day` 预测模型

课程问题要求构建一个数值预测模型 (numeric prediction model)，预测 `cvt_per_day`。

一个合理的 BA 建模流程可以这样写：

1. 明确目标变量：`cvt_per_day` 是连续数值，因此这是回归问题 (regression problem)。
2. 检查缺失值：看哪些字段缺失严重，判断是删除、填补还是单独标记。
3. 处理类别变量：例如 `Genres`, `mpaa`, `Import_id` 需要编码，可以用 one-hot encoding 或更合适的类别处理方法。
4. 处理数值变量：例如 `budget`, `boxoffice`, `imdb_votes` 可能分布很偏，可以考虑 log transform。
5. 做特征工程：例如内容年份可以转成内容年龄，首页位置可以转成曝光强弱信号。
6. 选择模型：先用 linear regression 做基线，再尝试 tree-based model，比如 random forest 或 gradient boosting。
7. 评估效果：用 RMSE、MAE、R-squared，并做 train/test split 或 cross validation。
8. 解释结果：找出哪些变量最能解释观看表现，而不是只报告分数。

### 缺失值处理思路

缺失值不要机械处理。不同字段的缺失含义可能不同：

- `budget` 缺失：可能是小成本或资料不全，不一定等于 0。
- `boxoffice` 缺失：可能没有院线上映，也可能数据未记录。
- `Awards` 缺失：可能没有获奖，也可能没有采集到。
- `Metacritic_score` 缺失：可能该内容没有足够媒体评分。

可以采用的处理方式：

- 数值字段用 median imputation，并增加 missing indicator。
- 类别字段把缺失当作 `Unknown` 类别。
- 缺失比例太高且业务解释弱的字段，可以考虑不纳入主模型。
- 对缺失本身有业务含义的字段，保留“是否缺失”这个信号。

## 3. Actual CPC：真实点击成本

SEM 里广告主设置的是最高愿意支付的点击价格 (Max CPC Bid)，但实际支付的价格通常不是自己的最高出价，而是由下一名广告的 Ad Rank 和自己的 Quality Score 决定。

课程给出的简化公式：

```text
Actual CPC = 下一名广告的 Ad Rank / 自己的 Quality Score + 0.01
```

关键含义：

- 你不一定支付自己的最高出价。
- 质量分 (Quality Score) 越高，实际 CPC 可能越低。
- 高质量广告可以用较低成本赢得更好位置。

例子里，一个广告主 Max CPC 只有 3 美元，但 Quality Score 是 8，Ad Rank 达到 24，实际 CPC 约为 2.51 美元；另一个广告主出价更高但质量分低，实际支付反而更贵。

BA takeaway：

- SEM 成本不是单纯由预算决定。
- 优化广告文案、关键词相关性、落地页体验，可以同时影响排名和成本。
- 分析投放时要同时看 bid、Quality Score、CTR、conversion 和 CPA。

## 4. SEM 效果衡量指标

课程用三个 campaign 比较 SEM 表现。核心指标如下：

```text
CTR = Clicks / Impressions
Conversion Rate = Conversions / Clicks
CPA = Total Cost / Conversions
Total Cost = Clicks x Cost per Click
```

指标解释：

| 指标 | 中文理解 | 用来回答的问题 |
| --- | --- | --- |
| Impressions | 展示次数 | 广告被用户看到多少次 |
| Clicks | 点击次数 | 有多少用户点进来 |
| CTR | 点击率 | 广告是否吸引用户点击 |
| CPC | 单次点击成本 | 买一次点击要多少钱 |
| Conversions | 转化次数 | 用户是否完成目标动作 |
| Conversion Rate | 转化率 | 点击进来的人有多少真的转化 |
| CPA | 单次获客成本 | 获得一个转化要花多少钱 |

课程中的三个 campaign 可以这样判断：

- Campaign A：CTR 较低，但 CPA 最低，获客成本更好。
- Campaign B：CTR 高、转化数高，但 CPA 高于 A。
- Campaign C：展示和点击更多，但转化率较低，CPA 最高。

所以“哪个最好”取决于业务目标：

- 如果目标是最低获客成本，A 更好。
- 如果目标是更多转化量且预算可接受，B 可以考虑。
- 如果 C 的 CPA 高且转化率低，需要检查关键词、受众或落地页是否不匹配。

## 5. A/B Testing 与两比例 Z 检验

课程给了一个面试型问题：广告 A 有 30000 次展示，CTR 是 10%；广告 B 有 20000 次展示，CTR 是 9.9%。A 看起来更高，但问题是：这个差异是否 statistically significant？

这里比较的是两个比例 (proportion)，所以可以用 two-proportion Z-test。

已知：

```text
n1 = 30000, p1 = 0.100, clicks1 = 3000
n2 = 20000, p2 = 0.099, clicks2 = 1998
```

合并比例 (pooled proportion)：

```text
p = (n1 x p1 + n2 x p2) / (n1 + n2)
```

Z 值：

```text
Z = (p1 - p2) / sqrt(p x (1 - p) x (1/n1 + 1/n2))
```

BA 需要表达清楚：

- 不能只看 10% 大于 9.9%，还要看样本量和统计显著性。
- 样本越大，小差异越可能显著；但显著不一定代表业务上重要。
- 面试中要同时说 statistical significance 和 practical significance。
- 如果只关心 A 是否高于 B，可以是一尾检验；如果只问是否不同，可以是双尾检验。

## 6. SEM 后台界面怎么读

课程展示了广告后台界面，常见字段包括：

- Search term：用户实际搜索词。
- Match type：匹配类型，例如 exact、phrase、broad。
- Campaign / Ad group：广告活动和广告组。
- Clicks / Impressions / CTR：曝光和点击表现。
- Avg CPC / Cost：点击成本和总成本。
- Avg Pos：平均排名位置。
- Conv / Cost per conv / Conv rate：转化、单次转化成本和转化率。

读后台时不要只看一个指标。一个关键词 CTR 高但 conversion rate 低，可能说明它吸引点击但意图不准；一个关键词 CPC 高但 CPA 低，可能仍然值得保留，因为它能带来高质量转化。

BA 可以按这个顺序排查：

1. 曝光是否足够：Impressions。
2. 用户是否愿意点：CTR。
3. 点击是否太贵：Avg CPC。
4. 是否产生业务结果：Conversions 和 Conversion Rate。
5. 转化是否划算：Cost per conv / CPA。

## 7. SEM vs SEO

课程比较了 SEM 和 SEO。两者不是谁替代谁，而是适合不同目标。

| 维度 | SEM | SEO |
| --- | --- | --- |
| 见效速度 | 通常 1-2 天可以看到结果 | 可能需要 2 周到 4 个月 |
| 控制力 | 可以快速开关预算和投放 | 流量更难精确控制 |
| 成本 | 每次点击和转化通常更贵 | 长期可能更 cost-effective |
| 曝光 | 付费位曝光受预算限制 | 自然结果更受用户信任 |
| 竞争 | 高竞争市场也能买曝光，但成本高 | 高竞争市场自然排名很难做 |
| 本地市场 | 更容易做 local targeting | 本地定向相对更难 |
| 适合场景 | 短期、高毛利、活动型 campaign | 长期、低毛利、内容沉淀型增长 |

BA takeaway：

- SEM 适合快速验证关键词、转化和市场需求。
- SEO 适合长期降低获客成本、积累内容资产。
- 很多业务会用 SEM 做短期拉动，用 SEO 做长期基本盘。

## 8. Customer Journey 与 Moment of Truth

客户旅程 (Customer Journey) 是用户从需求被触发，到搜索信息、比较方案、购买、使用、评价和分享的全过程。

课程重点讲了 Moment of Truth (MOT)，也就是用户和品牌、产品或服务接触并形成印象的关键瞬间。

### ZMOT：Zero Moment of Truth

ZMOT 是 Google 提出的概念，指用户在采取购买行动前，先在线上做研究的阶段。

例子：

- 搜索产品评价。
- 看手机测评。
- 比较价格。
- 看 YouTube 或社交媒体内容。
- 查附近门店或服务。

这和搜索引擎营销直接相关：用户的 ZMOT 很多发生在 search、review、video、social media 上。

### FMOT：First Moment of Truth

FMOT 是用户第一次真正面对产品并作出初步选择的瞬间。它可能发生在线下货架，也可能发生在线上商品页。

关键点：

- 用户通常在很短时间内形成判断。
- 页面设计、标题、价格、评价、图片和品牌信任都会影响选择。
- 对电商来说，商品列表页和详情页就是典型 FMOT 场景。

### SMOT：Second Moment of Truth

SMOT 是用户购买并实际使用产品后的体验时刻。

如果体验符合承诺，用户可能：

- 复购。
- 留好评。
- 推荐给朋友。
- 在社交媒体分享。

如果体验不符合承诺，前面的广告和转化可能会变成负面口碑。

## 9. Digital Marketing 与渠道

数字营销 (Digital Marketing) 指使用数字技术推广产品或服务。课程提到互联网、手机、展示广告和其他数字媒介。

常见方法包括：

- SEO：自然搜索优化。
- SEM：付费搜索广告。
- Content Marketing：内容营销。
- Email Marketing：邮件营销。
- Social Media Marketing：社交媒体营销。
- Display Advertising：展示广告。
- Mobile Apps：移动应用渠道。
- PR：公共关系和传播。

### Digital Marketing Vehicles

课程提到几个渠道类型：

1. Organic：自然流量，例如用户直接输入网址、收藏访问，或通过非付费方式自然来到网站。
2. SEM：来自外部搜索引擎的付费广告流量。
3. Email：通过邮件发送商业信息，用于广告、销售、关系维护、品牌认知或客户留存。

BA 视角下，每个渠道都要回答：

- 流量从哪里来？
- 用户意图是什么？
- 成本是多少？
- 转化质量如何？
- 是否能持续扩大？

## 10. BA 视角总结

这节课的主线可以概括成：

```text
数据 -> 模型 -> 投放指标 -> 实验判断 -> 客户旅程 -> 渠道策略
```

需要掌握的分析逻辑：

- 用数据集定义业务问题：预测什么、为什么预测、结果怎么用。
- 用 SEM 指标拆解投放表现：从 impression 到 click，再到 conversion 和 CPA。
- 用统计检验判断 A/B test 差异，而不是凭肉眼比较百分比。
- 用 customer journey 理解用户为什么搜索、为什么点击、为什么购买。
- 用渠道视角比较 SEM、SEO、organic、email 等不同流量来源的价值。

## 复习问题

1. `cvt_per_day` 为什么适合作为视频内容受欢迎程度的目标变量？
2. 处理 `budget`、`boxoffice` 这类缺失值时，为什么不能简单全部填 0？
3. Actual CPC 为什么可能低于自己的 Max CPC？
4. CTR 高但 CPA 也高，说明可能发生了什么？
5. A/B test 中 10% 和 9.9% 的差异，为什么不能直接说 A 更好？
6. SEM 和 SEO 分别适合什么业务场景？
7. ZMOT、FMOT、SMOT 分别对应客户旅程中的什么阶段？
8. Email marketing 为什么既可以用于获客，也可以用于留存？

## 术语表

| 术语 | 中文解释 |
| --- | --- |
| Data Driven Marketing | 数据驱动营销，用数据指导营销决策 |
| `cvt_per_day` | 每日累计观看时长，本课数据集的目标变量 |
| Regression | 回归，用来预测连续数值 |
| Feature Engineering | 特征工程，把原始字段加工成模型可用变量 |
| Actual CPC | 实际点击成本 |
| CTR | 点击率，Clicks / Impressions |
| Conversion Rate | 转化率，Conversions / Clicks |
| CPA | 单次获客成本，Total Cost / Conversions |
| A/B Testing | A/B 测试，对比两个版本效果 |
| Two-proportion Z-test | 两比例 Z 检验，用于比较两个比例指标 |
| Customer Journey | 客户旅程 |
| ZMOT | Zero Moment of Truth，用户购买前在线研究阶段 |
| FMOT | First Moment of Truth，用户第一次面对产品并形成选择的时刻 |
| SMOT | Second Moment of Truth，用户购买并使用后的体验时刻 |
| Digital Marketing | 数字营销 |
| Organic Traffic | 自然流量 |
| Email Marketing | 邮件营销 |
