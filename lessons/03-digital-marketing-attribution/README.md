# 第 03 课 - 数字营销渠道、预算分配与归因模型

日期：2026-07-08  
来源：BA track 第三节课 PDF，主题是 Digital Marketing Vehicles 和 Attribution Model  
整理方式：中文解释为主，保留关键英文术语；不是逐页照搬 PDF

## 一句话总结

这节课把数字营销渠道继续拆细：先比较 display ads、social media、affiliate、comparison shopping engine 等渠道的商业机制，再引出营销归因模型 (Attribution Model)，说明 BA 如何判断“哪个触点真正推动了转化”。

## 快速目录

1. 数字营销渠道在客户旅程中的位置
2. 展示广告 (Display Ads)
3. 广告位交易与竞价关系
4. 社交媒体营销 (Social Media Marketing)
5. YouTube 与平台型广告位
6. 联盟营销 (Affiliate Marketing)
7. 比价购物引擎 (Comparison Shopping Engine)
8. 渠道预算分配与边际收益递减
9. 营销归因 (Marketing Attribution) 的核心问题
10. 常见归因模型
11. 机器学习归因 (Algorithmic Attribution)
12. BA 视角总结

## 核心概念索引

- 数字营销渠道 (Digital Marketing Vehicles)：用户在线上接触品牌、广告或商品的不同入口。
- 展示广告 (Display Advertising / Display Ads)：网站或 App 中以 banner、文字、图片、视频、音频等形式展示的广告。
- 广告位卖方 (Ad Seller / Publisher)：拥有广告位的媒体、网站、平台或内容方。
- 广告中介 (Ad Broker / Ad Network / Platform)：连接广告位卖方和广告主的中间平台，例如 Google Ads、display ads broker、YouTube、Twitter 等。
- 广告主 (Advertiser / Ad Buyer)：购买广告曝光、点击或转化机会的公司。
- 竞价 (Bidding)：多个广告主争夺广告位时出价和竞争的过程。
- 社交媒体营销 (Social Media Marketing)：通过社交媒体平台推广产品或服务。
- 联盟营销 (Affiliate Marketing)：商家按访问、点击、订单或客户等结果向 affiliate 支付佣金的绩效型营销。
- 比价购物引擎 (Comparison Shopping Engine)：聚合多个零售商商品，帮助用户按价格、功能、评论等比较产品的平台。
- 边际收益递减 (Diminishing Return)：当某渠道投入增加时，每新增一元预算带来的新增收入可能下降。
- 营销归因 (Marketing Attribution)：量化每个营销触点对转化的贡献。
- 单触点归因 (Single Source Attribution / Single Touch Attribution)：把全部 credit 分给一个触点，常见如 last touch。
- 分数归因 (Fractional Attribution)：把 credit 按规则分给多个触点，例如 equal weights、position-based、curve model。
- 机器学习归因 (Algorithmic / Probabilistic Attribution)：用统计模型或机器学习估计不同触点对转化概率的影响。
- 转化路径 (Conversion Path)：用户从第一次接触广告到最终购买或转化之间的触点序列。

## 1. 数字营销渠道在客户旅程中的位置

上一节课已经讲到客户旅程 (Customer Journey)：用户从搜索、比较、访问网站、查看商品页，到结账、支付、配送和售后，会经历多个触点。

这节课先用电商流程串起来：

```text
Product Search -> E-Commerce Website -> Product Page -> Checkout & Payment -> Shipping
```

每一步都对应不同的业务优化点：

| 阶段 | 关键优化点 | BA 关注 |
| --- | --- | --- |
| Product Search | Good SEO | 用户能不能找到产品 |
| E-Commerce Website | Fast loading, simple navigation | 进入网站后是否顺畅 |
| Product Page | Description, images, videos, stock availability, price | 商品页是否能说服用户 |
| Checkout & Payment | Speed, ease, convenient payment | 结账是否产生流失 |
| Shipping | Free shipping, return policy | 下单后是否增强信任 |

这说明数字营销不只是“把人拉进来”，还包括用户进入网站后是否能继续推进到购买。BA 在分析营销效果时，不能只看广告点击，还要把后续页面体验和转化路径一起看。

## 2. 展示广告 (Display Ads)

展示广告 (Display Ads) 是出现在网站或 App 上的广告，可以是：

- banner。
- 文字广告。
- 图片广告。
- 视频广告。
- 音频广告。
- 其他互动广告格式。

Display advertising 的主要目的通常是向网站访客传递一般广告信息和品牌信息。它不一定像 SEM 那样强依赖用户主动搜索意图，而是通过媒体页面、内容场景或用户画像把广告展示给潜在用户。

BA 需要区分：

- Search ads 更接近“用户主动表达需求”。
- Display ads 更接近“品牌曝光、再营销或兴趣触达”。
- Display 的效果不应只用最后点击来判断，因为它可能影响的是认知和未来转化。

## 3. 广告位交易与竞价关系

课程用 SEM、Display、YouTube、Social Media 等渠道说明广告位背后的三方关系：

```text
广告位卖方 / 平台 -> 中介或广告平台 -> 广告主
```

不同渠道的形式不同，但商业逻辑相似：

| 渠道 | 广告位卖方 | 中介 / 平台 | 广告主 |
| --- | --- | --- | --- |
| SEM | Google 搜索结果页广告位 | Google Ads | Bank of America、Citi、Wells Fargo 等 |
| Display | 网站或 App 广告位 | Display ads broker / 广告交易平台 | 购买展示广告的公司 |
| YouTube | 视频平台和内容位置 | YouTube / Google Ads | 视频广告主 |
| Social Media | Twitter、Facebook 等社交平台广告位 | 平台广告系统 | 社交媒体广告主 |

共同点是：广告位有限，广告主需要通过竞价 (Bidding) 获得展示机会。竞价结果不只影响是否展示，也会影响成本、位置和可见度。

## 4. 社交媒体营销 (Social Media Marketing)

社交媒体营销 (Social Media Marketing) 是通过社交平台和社交网站推广产品或服务。课程里用 Twitter 上的 promoted post 举例：用户在信息流中看到品牌广告，广告通常以普通内容相似的形式出现，但带有 promoted 标识。

社交媒体广告的特点：

- 依赖用户兴趣、社交关系和内容上下文。
- 可以做精准定向，例如地域、兴趣、人群属性。
- 可以带来曝光、互动、点击和转化。
- 用户反馈更明显，例如转发、点赞、评论、负面回应。

BA 不能只看社交广告是否有点击，还要看它在 funnel 中承担什么角色：

- 是 brand awareness？
- 是把用户导向落地页？
- 是促销活动转化？
- 是再营销 (retargeting)？
- 是否带来长期用户关系或口碑？

## 5. YouTube 与平台型广告位

YouTube 这类平台既是内容平台，也是广告平台。课程把 YouTube 放在类似广告市场的结构中理解：

```text
内容/平台提供广告位 -> YouTube 作为中介和平台 -> 广告主购买视频广告位
```

这类广告的特殊点是：

- 广告可能出现在视频前、中、后，或页面推荐区。
- 用户注意力和跳过行为很重要。
- 视频广告既可以做品牌曝光，也可以导流到网站。
- 衡量指标可能包括 impression、view、view-through rate、click、conversion。

BA 分析视频广告时要注意：视频曝光本身可能影响后续搜索和购买，即使用户当下没有点击。

## 6. 联盟营销 (Affiliate Marketing)

联盟营销 (Affiliate Marketing) 是一种绩效型营销。企业奖励 affiliate，因为 affiliate 通过自己的营销努力带来了访问者或客户。

课程用返现网站举例：

```text
Customer -> Mr. Rebates -> Walmart
```

假设：

- 顾客通过返现网站进入 Walmart。
- 顾客在 Walmart 消费 100 美元。
- Walmart 给 Mr. Rebates 5 美元佣金。
- Mr. Rebates 返给顾客 4 美元。
- 顾客实际花费约 96 美元获得 100 美元商品价值。

这个机制背后的问题是：Walmart 是否应该公开告诉用户“每消费 100 美元返 5 美元”？这不是简单的数学问题，而是渠道策略问题。

需要考虑：

- 公开返现可能提升短期转化。
- 但也可能训练用户等折扣或通过 affiliate 才购买。
- 如果本来会直接购买的用户被 affiliate 抢走，商家反而多付佣金。
- 需要区分增量转化 (incremental conversion) 和原本就会发生的转化。

BA takeaway：Affiliate 的核心不是“有没有订单”，而是“这些订单是否真的是 affiliate 带来的新增价值”。

## 7. 比价购物引擎 (Comparison Shopping Engine)

比价购物引擎 (Comparison Shopping Engine) 有时也叫 price comparison website、price analysis tool、comparison shopping agent 或 shopbot。

它的作用是帮助用户按以下标准筛选和比较商品：

- 价格。
- 功能。
- 评论。
- 品牌。
- 商家。
- 库存和配送信息。

很多比价平台并不直接销售商品，而是聚合不同零售商的商品列表，通过 affiliate marketing agreements、广告位或导流费用获得收入。

课程提出一个关键问题：Sort by "default"，什么是 default？

这提醒 BA：默认排序看起来是中立的，但可能受到商业因素影响，例如：

- 谁付了更高广告费。
- 哪个商家佣金更高。
- 哪个商品转化率更高。
- 平台希望优化 GMV、利润还是用户体验。

所以在分析列表页和推荐排序时，要问清楚排序目标函数是什么。

## 8. 渠道预算分配与边际收益递减

课程给了一个预算分配案例。上个月总预算是 138,000 美元，不同渠道的花费、收入和成本表现如下：

| Channel | Total spend | Revenue generated | Cost per $1 of closed business |
| --- | ---: | ---: | ---: |
| Social | $8,000 | $8,542 | $0.94 |
| SEM | $20,000 | $76,522 | $0.26 |
| Email | $30,000 | $98,453 | $0.30 |
| Call center | $50,000 | $85,005 | $0.59 |
| Display | $30,000 | $95,234 | $0.32 |

下个月预算翻倍到 276,000 美元，问题是如何分配预算以最大化 revenue。

直觉上可能会把钱投给成本最低的 SEM、Email 或 Display。但课程提醒要考虑边际收益递减 (Diminishing Return)：

- 原来 SEM 投 20,000 美元效果好，不代表投 200,000 美元仍然保持同样效率。
- 每个渠道有规模上限、受众容量、疲劳效应和竞争成本。
- 历史平均 ROI 不等于未来边际 ROI。
- 预算优化应看新增预算带来的新增收入，而不是只看平均成本。

BA 应该用 marginal ROAS 或 response curve 的思路评估预算：

```text
新增预算 -> 新增曝光/点击/触达 -> 新增转化 -> 新增收入
```

## 9. 营销归因 (Marketing Attribution) 的核心问题

营销归因 (Marketing Attribution) 的目标是量化每个广告曝光或营销触点对用户购买决策的影响。

用户转化往往不是由一次触点造成的，而是一个 multi-touch path：

```text
Day 1: Click Email Promotion Link -> no conversion
Day 1: Click TripAdvisor Link -> no conversion
Day 8: Click Paid Search Link -> conversion
```

问题是：

- 哪个 campaign 应该获得 conversion 的 credit？
- 每个 touch point 应该被分配多少价值？
- 如果只看最后一次点击，会不会低估前面的 email 或 review site？

归因数据可以帮助营销团队：

- 优化 media spend。
- 比较 paid search、organic search、email、affiliate、display、social media 等渠道价值。
- 分析哪些 media placements 最 cost-effective。
- 用 ROAS、CPL 等指标评估历史 campaign 并规划未来 campaign。

## 10. 常见归因模型

### 10.1 单触点归因 (Single Source Attribution)

单触点归因也叫 single touch attribution。它把全部 credit 分配给一个事件，例如最后一次触点。

常见例子：

- Last touch attribution：把转化全部归给最后触点。
- First touch attribution：把转化全部归给第一次触点。
- 3-day last touch：只在特定窗口内把转化归给最近触点。

优点：

- 简单。
- 容易解释。
- 数据要求较低。

问题：

- 过度高估最后触点。
- 低估早期触点对认知和兴趣的影响。
- 容易导致预算错误倾斜。

课程还提醒：

- 频繁 email 可能带来更多点击，但长期会让用户反感。
- 一次性折扣可能短期提高转化，但伤害后续转化。
- 新用户往往需要多个 touchpoints 才转化，不能因此认为初始触点无效。

### 10.2 分数归因 (Fractional Attribution)

分数归因会把 credit 分配给多个触点。它包括：

- equal weights。
- multi-touch model。
- curve model。
- position-based model。

课程图示里，一个路径可以分成：

```text
first touch 40% + middle touches 20% + last touch 40%
```

这种方法比单触点更合理，因为它承认多个触点共同影响转化。

但核心问题是：权重怎么定？

如果 40/20/40 是人为规则，它可能更符合直觉，但不一定符合真实数据。不同业务、不同产品、不同购买周期下，合理权重可能完全不同。

### 10.3 数据驱动归因 (Data-driven Attribution)

数据驱动归因希望用真实用户路径数据来估计触点价值，而不是完全靠人为规则。

它会关注：

- 用户在哪些阶段接触品牌。
- 哪些触点更常出现在转化路径中。
- 哪些触点也大量出现在未转化路径中。
- 不同顺序、组合、时间间隔对转化概率的影响。

BA 角度下，data-driven attribution 的价值在于：它把“渠道争功劳”转成“路径贡献建模”问题。

## 11. 机器学习归因 (Algorithmic Attribution)

课程最后讲 machine learning attribution，也叫 algorithmic attribution 或 probabilistic attribution。

核心思路：

```text
用户触点路径 -> 构造特征 X -> 是否购买 y -> 建模 -> 用特征权重估计触点贡献
```

### 11.1 定义目标变量

目标变量是二分类：

```text
y = 1：用户购买
y = 0：用户没有购买
```

这和上一课 Amazon Prime Video 的 `cvt_per_day` 回归问题不同。这里的归因模型更像 conversion classification。

### 11.2 构造特征

假设有 5 种 marketing actions：

1. Google search / SEM。
2. Email。
3. Display ad。
4. Social。
5. Comparison engine。

如果最多考虑 5 个位置，就可以构造：

```text
5 actions x 5 positions = 25 binary features
```

例如：

- Feature 1：Action 1 是否出现在第 1 位。
- Feature 2：Action 1 是否出现在第 2 位。
- Feature 5：Action 1 是否出现在第 5 位。
- Feature 6：Action 2 是否出现在第 1 位。
- Feature 25：Action 5 是否出现在第 5 位。

每个特征是 0/1：

```text
如果该 action 出现在该位置，feature = 1
否则 feature = 0
```

### 11.3 用户路径示例

如果顾客路径是：

```text
Customer 1: Action 1 (SEM) 顺序 1 -> Action 2 (email) 顺序 2 -> buy
```

那么可以编码为：

```text
X = (1, 0, 0, 1)
y = 1
```

如果顾客路径是：

```text
Customer 2: Action 2 顺序 1 -> Action 1 顺序 2 -> did not buy
```

则：

```text
X = (0, 1, 1, 0)
y = 0
```

这类编码的重点是：不仅记录用户接触了什么渠道，还记录渠道出现的顺序。

### 11.4 用模型权重做归因

课程用 logistic regression 举例。模型训练后，每个 feature 会有一个系数：

```text
K1, K2, ..., K25
```

如果要算某个 marketing vehicle (MV) 的归因权重，可以把它对应位置上的系数加总，再除以所有系数总和。

例如：

```text
Attribution of MV1 = (K1 + K2 + ... + K5) / sum(Ki)
Attribution of MV2 = (K6 + K7 + ... + K10) / sum(Ki)
...
Attribution of MV5 = (K21 + K22 + ... + K25) / sum(Ki)
```

课程最后展示了更复杂的 12-action 归因例子，每个触点可以得到不同百分比，例如 11%、6%、4%、39%、6%、11%、23% 等。这说明模型归因可以比固定规则更细，尤其适合多渠道、多路径的转化场景。

注意：真实业务中不能机械相信系数。需要检查数据质量、样本偏差、渠道共线性、时间窗口和因果解释边界。

## BA 视角总结

这节课的主线是：

```text
营销渠道 -> 广告位竞价 -> 渠道预算 -> 多触点转化 -> 归因模型 -> 预算优化
```

BA 需要掌握的不是某一个公式，而是分析逻辑：

- 渠道不是孤立的，用户可能先看 display，再看 social，再搜索，最后购买。
- 预算不能只按历史平均 ROI 分配，要看边际收益递减。
- Last touch 简单但可能误导预算。
- Fractional attribution 更全面，但权重规则可能主观。
- Machine learning attribution 可以利用真实路径数据，但要谨慎解释模型权重。
- 最终目标是把 attribution 结果转成业务动作，例如调预算、改投放组合、优化 customer journey。

如果要在实际项目中落地，BA 可以按这个流程做：

1. 明确转化目标：购买、注册、预约、付费还是留资。
2. 定义 touchpoint 和 attribution window。
3. 收集用户路径数据。
4. 区分转化路径和未转化路径。
5. 选择归因方法：last touch、position-based、data-driven 或 machine learning。
6. 输出渠道贡献、边际 ROI 和预算建议。
7. 用实验或后续数据验证调整是否有效。

## 复习问题

1. Display Ads 和 SEM 的用户意图有什么区别？
2. 为什么 social media ads 不能只用点击数判断效果？
3. Affiliate Marketing 中，为什么要区分新增转化和原本就会发生的转化？
4. Comparison Shopping Engine 的默认排序为什么可能不是完全中立的？
5. 如果下个月预算翻倍，为什么不能简单把钱全部投给历史 ROI 最高的渠道？
6. Marketing Attribution 要解决的核心问题是什么？
7. Last touch attribution 有什么优点和缺点？
8. Fractional attribution 的核心难点是什么？
9. 机器学习归因中，为什么要把 action 和 position 组合成 binary features？
10. Logistic regression 的系数在归因场景中可以如何解释？有哪些风险？

## 术语表

| 术语 | 英文 | 中文解释 |
| --- | --- | --- |
| 数字营销渠道 | Digital Marketing Vehicles | 用户在线接触品牌、广告或商品的不同入口 |
| 展示广告 | Display Ads / Display Advertising | 网站或 App 中的 banner、图片、视频等广告 |
| 广告位卖方 | Ad Seller / Publisher | 拥有广告位的媒体、网站或平台 |
| 广告主 | Advertiser / Ad Buyer | 购买广告曝光、点击或转化机会的企业 |
| 广告中介 | Ad Broker / Ad Network | 连接广告位供给和广告需求的平台或中介 |
| 竞价 | Bidding | 广告主为争夺广告位而出价竞争 |
| 社交媒体营销 | Social Media Marketing | 通过社交平台推广产品或服务 |
| 联盟营销 | Affiliate Marketing | 按访问、订单、客户等结果给 affiliate 支付佣金的营销方式 |
| 比价购物引擎 | Comparison Shopping Engine | 聚合多家零售商商品并支持价格、功能、评论比较的平台 |
| 边际收益递减 | Diminishing Return | 投入增加后，每新增单位投入带来的新增收益下降 |
| 营销归因 | Marketing Attribution | 判断不同营销触点对转化贡献的分析方法 |
| 转化路径 | Conversion Path | 用户从接触广告到转化之间的触点序列 |
| 单触点归因 | Single Source Attribution | 把全部转化 credit 分给一个触点 |
| 最后触点归因 | Last Touch Attribution | 把全部 credit 分给转化前最后一个触点 |
| 分数归因 | Fractional Attribution | 把 credit 分配给多个触点 |
| 位置归因 | Position-based Attribution | 根据触点位置分配不同权重的归因方法 |
| 数据驱动归因 | Data-driven Attribution | 用真实路径数据估计触点贡献 |
| 机器学习归因 | Algorithmic / Probabilistic Attribution | 用统计模型或机器学习估计触点对转化概率的影响 |
| 营销触点 | Touchpoint | 用户与品牌、广告或渠道的一次接触 |
| 广告支出回报 | Return on Ad Spend, ROAS | 广告收入与广告成本的关系 |
| 单条线索成本 | Cost Per Lead, CPL | 获得一个潜在线索所需成本 |
