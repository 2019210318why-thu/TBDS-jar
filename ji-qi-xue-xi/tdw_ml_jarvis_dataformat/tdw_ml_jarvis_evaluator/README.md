# 模型评估

* [Binary Evaluator](./#1-binary-evaluator)
* [Multi Evaluator](./#2-multi-evaluator)
* [Regrssion Evaluator](./#3-regression-evaluator)

## 1. Binary Evaluator

* **算法说明**

  用于评估二分类算法的预测结果。预测的结果必须是0~1的概率值，评估结果包括混淆矩阵和ROC/PR auc等指标。

* **输入**
  * 数据形式：[Dense](../tdw_ml_jarvis_dataformat.md#21-dense数据格式)
  * 格式：\| label \| score \|
  * label:样本真实的label，通过**参数**中的**标签列**指定
  * score:预测值，模型预测的0~1.0之间的概率值所在的列，通过**参数**中的**预测列**指定
* **参数**
  * 标签列\(labelCol\)：标签所在列，从0开始计数
  * 预测列\(scoreCol\)：预测值所在列，从0开始计数
  * 预测阈值
    * 范围：0~1.0的值，预测值大于该阈值，则预测为1；否则为0
    * 说明：在真实的CTR预估中，预测的概率值都比较小，因此要视真实情况设定该参数
* **输出**
  * True class和Hypothesized class的混淆矩阵
  * ROC/PR AUC值

## 2. Multi Evaluator

* **算法说明**

  用于评估多分类算法的预测结果。样本的预测结果是模型预测的类别；模块将会统计类别真实类列和预测类别的混淆矩阵和各个类别的f1，precision，recall。

* **输入**
  * 数据形式：[Dense](../tdw_ml_jarvis_dataformat.md#21-dense数据格式) 
  * 格式：\| label \| predict \|
  * label:样本真实的label，保存的是样本真实的类别，通过**参数**中的**标签列**指定
  * score:预测值，保存的是模型预测的类别，通过**参数**中的**预测列**指定
* **参数**
  * 标签列\(labelCol\)：标签所在列，从0开始计数
  * 预测列\(predictCol\)：预测的类别所在列，从0开始计数
* **输出**
  * True class和Hypothesized class的混淆矩阵
  * 不同类别的f1, precision, recall。

## 3. Regression Evaluator

* **算法说明**

  用于评估回归算法的预测结果。输入的是真实的值和模型预测值；模型将计算MAE/MSE/rMAE/R2等指标。

* **输入**
  * 数据形式：[Dense](../tdw_ml_jarvis_dataformat.md#21-dense数据格式)
  * 格式：\| label \| predict \|
  * 说明：以空格连接各字段
  * label:样本真实的label，通过**参数**中的**标签列**指定
  * predict:预测值，通过**参数**中的**预测列**指定
* **参数**
  * 标签列\(labelCol\)： 标签所在列，从0开始计数
  * 预测列\(scoreCol\)：预测值所在列，从0开始计数
* **输出**
  * MAE/MSE/rMAE/R2等指标

