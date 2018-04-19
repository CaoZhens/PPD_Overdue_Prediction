## PPD_Overdue_Prediction 拍拍贷贷款逾期概率预测
### [描述 & 数据 & 目标](./readme/readme_part1.md)
### 处理流程摘要
#### 数据概述（目前仅考虑初赛轮数据）
> 训练集: 3w 227（226+1）
测试集: 2w 226
trainset&testset的索引列idx无交集，因此可以较容易concat(axis=1)到一起处理；只需要额外记录训练集和测试集的idx即可

#### 数据加载
> 有两类数据： 
结构化数据（有唯一索引） 
用户日志数据（时间点+登陆操作／修改信息操作）

**数据加载中的数据清洗**: 包括缺失值转换／字段的简单格式化（日期格式化／字符串格式化）
- **字段的简单格式化**
  - 日期格式化：直接将日期特征转换为日期类型，便于后续处理
  - 字符串格式化：例如省／城市字端中直接去掉结尾的“省”／“市”
- **缺失值分析&转换**
有些特征虽然有取值，但实际上应该表示为缺失值形式。比如：省／城市字段中的“不详”；  数值字段中的-1可以在数据加载时直接进行缺失值转换


#### 数据转换（主要是日志数据）
（利用日志数据**构造日志类特征**）
加载时将日志时间点&借贷时间点处理成时间差形式
计算最早操作距今时间差／最晚操作距今时间差／操作次数／操作天数count distinct day/
操作频次（利用操作天数／最早操作距今时间差计算得到 ）
不同操作的次数

#### 数据摘要（略）
#### 数据清洗
**根据数据摘要确定数据清洗的规则**
**特征列分类**
不同类型的特征（列）分别处理，共七大类
education ／ listinginfo ／ socialnetwork ／ thirdparty ／ userinfo ／ webloginfo ／ userupdate

Date     - 日期类特征(详见github的总结)
Category - 逻辑类特征
  有额外信息的Category特征（如城市类特征）
    城市类特征: 
      省份：可保留直接做独热编码
      城市：取值过多，关联外部数据转化为经纬度+城市等级（城市名不要了）—— 也属于一种逻辑特征数值化的方式
  其余特征：独热编码
Numerical:
  **相似数值特征按统计量合并** ColS_summary（一般针对含义相似or统计量相似的特征： 如thirdparty系列；城市系列2 4 8）

特征选择：
  1. **基于统计值的特征选择**： 删除稀疏特征 & 共线性特征

缺失值填充
特征归一化

#### 单模型训练及调参

#### 多模型融合

1. skf with no shuffle
lr with default parameters
par = {'penalty': 'l2', 'C': 0.003, 'class_weight': 'balanced', 'solver': 'sag'}
ValidSet Score with 10-folds: 0.7551 +/- 0.0202

lr after hyperopt
{'C': 0.0026510157478321474}
ValidSet Score with 10-folds: 0.7553 +/- 0.0202

TestSet
auc: 0.770800353744

2. skf with shuffle
ValidSet Score with 10-folds: 0.7685 +/- 0.0229
{'C': 0.003290797431595955}
ValidSet Score with 10-folds: 0.7685 +/- 0.0229
auc: 0.77084207963



{'colsample_bylevel': 0.07, 'eta': 0.05, 'max_depth': 3, 'lambda': 50, 'min_child_weight': 1.5, 'gamma': 0.2}
ValidSet Score with 10-folds: 0.7717 +/- 0.0178
