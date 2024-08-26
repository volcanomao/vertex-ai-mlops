![tracker](https://us-central1-vertex-ai-mlops-369716.cloudfunctions.net/pixel-tracking?path=statmike%2Fvertex-ai-mlops%2FApplied+Forecasting&file=readme.md)
<!--- header table --->
<table align="left">     
  <td style="text-align: center">
    <a href="https://github.com/statmike/vertex-ai-mlops/blob/main/Applied%20Forecasting/readme.md">
      <img src="https://cloud.google.com/ml-engine/images/github-logo-32px.png" alt="GitHub logo">
      <br>View on<br>GitHub
    </a>
  </td>
</table><br/><br/><br/><br/>

---
# /Applied Forecasting/readme.md

**系列介绍：应用预测**

这个系列探讨了使用 Vertex AI、BigQuery ML 和其他开源框架进行预测。预测包括随着时间的推移对某个测量值进行跟踪，探索趋势、季节性影响（年、月、日等）、节假日和特殊事件，希望利用这些见解来预测近期的未来。一些方法还结合了影响需求的可观察测量值，以了解关系并使预测更加准确。

**数据来源：纽约市的 Citibike 租赁**

这个系列将使用纽约市的 Citibike 租赁数据。将选择中央公园附近的自行车站，并随着时间的推移跟踪从这些站点出发的每日自行车行程数量。这将说明一些常见的预测问题，因为随着时间的推移会引入新的站点，一些站点仅有最近几个月或几周的数据。这些数据可以在 BigQuery 公共数据集找到：`bigquery-public-data.new_york.citibike_trips`。

<table style='text-align:center;vertical-align:middle' width="75%" cellpadding="1" cellspacing="0">
    <tr>
        <th colspan='2'>Citibike 站点</th>
    </tr>
    <tr>
        <td>
            <a href="https://www.google.com/maps/search/central+park+citibike+stations/@40.7794305,-73.9733652,14z" target="_blank">
                <img src="../architectures/notebooks/applied/forecasting/citibike_central_park.png" width="100%">
                <h4 align="center">中央公园站点</h4>
            </a>
        </td>
        <td>
            <img src="../architectures/notebooks/applied/forecasting/citibike_central_park_s_6_ave.jpg" width="75%">
            <h4 align="center">中央公园 S & 6th Avenue</h4>
        </td>
    </tr>
</table>

---
## 预测主题

**单变量预测**

核心上，预测是对未来时间点目标的预测。为此，仅需要两列：一列表示观察的时间点，另一列捕捉测量值。这两列构成了时间序列。仅使用这两列，可以选择的方法有：
- 基于回归的方法预测序列中的下一个值
- 多步回归模型预测未来时间范围内的每个值
- 使用参数化方法如 ARIMA
    - 建立一个公式，将未来表示为过去时间点的函数，具有之前时间点 (`p`)、差分 (`d`) 和移动平均 (`q`) 的参数
- 使用广义加性模型方法如 [Prophet](https://github.com/facebook/prophet)
    - 这种方法可以检测变更点

**预训练时间序列基础模型**

一种新的预测模型类别，预训练用于预测推理（概念上类似于文本的 LLMs）。这些模型已经训练好，接受时间序列作为输入并生成预测范围。

**多变量预测**

所有这些都是局部预测，意味着每个时间序列独立拟合。这些可以扩展以使用多变量预测中的协变量来包含更多信息。与时间相关的额外信息有助于理解随时间发生的变化。协变量可能是可以提前知道的信息，如折扣、节假日期间、促销和广告。也可能是直到实际发生时才知道的信息，如天气、交通和业务中断。

**全球预测**

除了协变量（甚至不需要协变量），还有一些方法可以跨多个时间序列进行学习，以期获得更大的泛化能力。这些称为全球预测技术，因为时间序列同时一起学习。

**更多主题**

预测项目中还会出现其他主题，包括：
- 分层预测。这些方法允许时间和/或分类的层次结构得以保留，使得预测值的聚合有意义。
- 针对新时间序列的冷启动或热启动预测（考虑新产品销售预测问题）。
- 使用动态时间规整等技术将时间序列聚类为行为类似的组。
- 更多！

**在线预测**

上述方法属于批量预测。由于在某个时间点使用训练数据并创建未来时间点的预测模型，这是最常见的预测方法，并适用于更多情况。可以批量生成预测，保存并根据业务应用程序（如仪表板）需要进行调用。

如果需要按需预测怎么办？
- 也许时间序列数量太多，无法持续训练，创建所需预测更容易
- 也许数据的速度要求即时训练和服务，以便做出更快的决策——如异常检测

Google Cloud [Time Series Insights API](https://cloud.google.com/timeseries-insights) 是满足这些场景的规模和速度要求的解决方案。还可以将本地预测方法构建到 Vertex AI 端点中，使用自定义预测例程（CPR）。下面给出了使用 Prophet 的 CPR 本地预测示例。

---
**前提条件**

- 环境设置：[00 - Setup.ipynb](../00%20-%20Setup/00%20-%20Environment%20Setup.ipynb)

## 笔记本：
这是为任何想要了解 GCP 预测方法并进行学习的人建议的审阅顺序列表。如果对某个特定笔记本感兴趣，也可以选择，如果有依赖关系，它们将在笔记本顶部的 **前提条件** 部分列出。

> 笔记本被设计为可编辑的，以便尝试其他数据源。相同的参数名称在笔记本之间使用，有助于在自定义数据源上尝试多种方法。

- 数据来源：
    - 1 - [BigQuery 时间序列预测数据审查和准备](./BigQuery%20Time%20Series%20Forecasting%20Data%20Review%20and%20Preparation.ipynb)
- BigQuery ML：单变量和多变量局部预测方法
    - 2 - [BQML 单变量预测与 ARIMA+](./BQML%20Univariate%20Forecasting%20with%20ARIMA+.ipynb)
    - 3 - [BQML 多变量预测与 ARIMA+ XREG](./BQML%20Multivariate%20Forecasting%20with%20ARIMA+%20XREG.ipynb)
    - 4 - [BQML 基于回归的预测](./BQML%20Regression%20Based%20Forecasting.ipynb)
    - [笔记 - BQML ARIMA+ 的粒度和缺失数据处理](./Notes%20-%20BQML%20ARIMA%2B%20Handling%20of%20Granularity%20and%20Missing%20Data.ipynb)
- 预训练预测模型：Google Research 时间序列基础模型（TimesFM）
    - [TimesFM - 时间序列基础模型](./TimesFM%20-%20Time%20Series%20Foundation%20Model.ipynb)
- Vertex AI AutoML：用于单变量或多变量的全球预测，包括分层预测
    - 5 - [Vertex AI AutoML 预测 - GCP 控制台（无代码）](./Vertex%20AI%20AutoML%20Forecasting%20-%20GCP%20Console%20(no%20code).ipynb)
    - 6 - [Vertex AI AutoML 预测 - Python 客户端](./Vertex%20AI%20AutoML%20Forecasting%20-%20Python%20client.ipynb)
    - 7 - [Vertex AI AutoML 预测 - 同时进行多个预测](./Vertex%20AI%20AutoML%20Forecasting%20-%20multiple%20simultaneously.ipynb)
- Vertex AI 其他：用于单变量或多变量的全球预测，包括分层预测
    - 8 - [Vertex AI Seq2Seq+ 预测 - Python 客户端](./Vertex%20AI%20Seq2Seq+%20Forecasting%20-%20Python%20client.ipynb)
    - 9 - [Vertex AI 时间序列融合变换器预测 - Python 客户端](./Vertex%20AI%20Temporal%20Fusion%20Transformer%20Forecasting%20-%20Python%20client.ipynb)
    - 10 - [Vertex AI 时间序列稠密编码器 - Python 客户端](./Vertex%20AI%20Time%20series%20Dense%20Encoder%20-%20Python%20client.ipynb)

	-	Vertex AI 自定义模型：使用 Prophet 进行单变量和多变量本地预测
	-	11 - Vertex AI 自定义模型 - Prophet - 笔记本中
	-	12 - Vertex AI 自定义模型 - Prophet - 使用自定义容器进行自定义作业
	-	Vertex AI 管道：全预测工作流的自动化
	-	13 - Vertex AI 管道 - BQML ARIMA+
	-	14 - Vertex AI 管道 - Prophet
	-	15 - Vertex AI 管道 - 使用 Kubeflow 管道进行预测比赛 (KFP)
	-	预测管道 欲了解更多详细的使用管道进行预测的起点，我强烈推荐我的同事 Jordan Totten 的这个仓库！
	-	高级工作流：
	-	Vertex AI 预测端点用于在线预测与 Prophet

## 笔记

	-	资源：
	-	价格优化示例，如：https://cloud.google.com/blog/products/ai-machine-learning/price-optimization-using-vertex-ai-forecast