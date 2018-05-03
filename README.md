## PPD_Overdue_Prediction 拍拍贷贷款逾期概率预测
### [描述 & 数据 & 目标](./readme/readme_part1.md)
### 处理流程摘要
#### 数据概述（目前仅考虑初赛轮数据）
> TrainSet: 3w 227(226+1)  
TestSet : 2w 226  
TrainSet&TestSet的索引列Idx无交集，因此可以较容易concat(axis=0)到一起处理；只需要额外记录训练集和测试集的idx即可

#### 数据加载
> 数据源分为两类:  
结构化数据（有唯一索引）  
用户日志数据（时间点+登陆操作／修改信息操作）

**数据加载中的数据清洗** 
> 主要包括缺失值转换／字段的简单格式化（日期格式化／字符串格式化）
- **缺失值分析&转换**  
有些特征虽然有取值，但实际上应该表示为缺失值形式。比如：省／城市字段中的“不详”  
数值字段中的-1可以在数据加载时直接进行缺失值转换  
- **字段的简单格式化**  
  - 日期格式化：直接将日期特征转换为日期类型，便于后续处理
  - 字符串格式化：例如省／城市字端中直接去掉结尾的“省”／“市”


#### 数据转换（主要针对非结构化数据如日志数据）
- **构造日志类特征**  
加载时将日志时间点&借贷时间点处理成时间差形式  
计算最早操作距今时间差／最晚操作距今时间差／操作次数／操作天数count distinct day/  
操作频次（利用操作天数／最早操作距今时间差计算得到 ）  
不同操作的次数  

#### 数据摘要（略）
#### 数据清洗
##### 数据清洗
**根据数据摘要确定数据清洗的规则**
> **特征列分类**  
不同类型的特征（列）分别处理，共七大类  
education ／ listinginfo ／ socialnetwork ／ thirdparty ／ userinfo ／ webloginfo ／ userupdate

- Date     - 日期类特征(详见github的总结)
- Category - 逻辑类特征  
  有额外信息的Category特征（如城市类特征）  
    城市类特征:   
      省份：可保留直接做独热编码  
      城市：取值过多，关联外部数据转化为经纬度+城市等级（城市名不要了）—— 也属于一种逻辑特征数值化的方式  
  其余特征：独热编码  
- Numerical - 数值类特征  
  **相似数值特征按统计量合并** ColS_summary（一般针对含义相似or统计量相似的特征： 如thirdparty系列；城市系列2 4 8）  

##### 特征选择
  1. **基于统计值的特征选择**： 删除稀疏特征 & 共线性特征

##### 缺失值填充
##### 特征归一化
> 标准归一化 & 正态归一化

#### 单模型训练及调参
**LR with StratifiedKFold(k=10)**  
Default Parameters  

|      模型参数      |      CV      | ValidSet avg AUC | TestSet AUC result |
|:-----------------:|:------------:|:----------------:|:------------------:|
| 'penalty': 'l2', 'C': 0.003, 'class_weight': 'balanced', 'solver': 'sag' | StratifiedKFold(shuffle=False) |  0.7551 +/- 0.0202  |  |
| 'penalty': 'l2', 'C': 0.003, 'class_weight': 'balanced', 'solver': 'sag' | StratifiedKFold(shuffle=True)  |  0.7685 +/- 0.0229  |  |

Tuning with HyperOpt  

|        最优参数       |      CV        | testSet avg AUC |
|:--------------------:|:---------------------------------:|:-------:|
| 'C': 0.002450953481265099 | StratifiedKFold(shuffle=False) |  0.7554 +/- 0.0202  |
| 'C': 0.003040794257300518 | StratifiedKFold(shuffle=True)  |  0.7685 +/- 0.0229  |

**XGBoost with StratifiedKFold(k=10)**  
Default Parameters  

|        模型参数        |      CV    | testSet avg AUC |
|:--------------------:|:-----------:|:---------------:|
| 'colsample_bylevel': 1, 'eta': 0.05, 'max_depth': 3, 'lambda': 1, 'min_child_weight': 1, 'gamma': 0     | StratifiedKFold(shuffle=False) |  0.5781 +/- 0.1568  |
| 'colsample_bylevel': 0.07, 'eta': 0.05, 'max_depth': 3, 'lambda': 50, 'min_child_weight': 1.5, 'gamma': 0.2   | StratifiedKFold(shuffle=False) |  0.7105 +/- 0.0330  |
| 'colsample_bylevel': 1, 'eta': 0.05, 'max_depth': 3, 'lambda': 1, 'min_child_weight': 1, 'gamma': 0     | StratifiedKFold(shuffle=True)  |  0.7666 +/- 0.0162  |
| 'colsample_bylevel': 0.07, 'eta': 0.05, 'max_depth': 3, 'lambda': 50, 'min_child_weight': 1.5, 'gamma': 0.2   | StratifiedKFold(shuffle=True)  |  0.7724 +/- 0.0179  |
| 'subsample': 0.8, 'colsample_bytree': 0.8, 'colsample_bylevel': 1, 'eta': 0.1, 'max_depth': 5, 'lambda': 1, 'min_child_weight': 1, 'gamma': 0 | StratifiedKFold(shuffle=True)  |  0.7667 +/- 0.0169  |

Tuning Parameters (by HyperOpt)  

|        模型参数        |      CV    | testSet avg AUC |
|:--------------------:|:-----------:|:---------------:|
| 'max_depth': 3,'subsample': 0.6,'min_child_weight': 2.77,'colsample_bytree': 0.7,'colsample_bylevel':0.1,'eta': 0.05 | StratifiedKFold(shuffle=False)  |  0.7131 +/- 0.0351   |
| 'max_depth': 3, 'subsample': 0.6, 'min_child_weight': 2.77, 'colsample_bytree': 0.7, 'colsample_bylevel':1,  'eta': 0.1  | StratifiedKFold(shuffle=True)   |   0.7722 +/- 0.0181  |

Tuning Parameters (by HyperOpt) with StratifiedKFold(shuffle=True)  

|        模型参数        |      CV    | testSet avg AUC |
|:--------------------:|:-----------:|:---------------:|
| 'max_depth': 3, 'subsample': 0.9, 'min_child_weight': 1.2, 'colsample_bytree': 0.2,'colsample_bylevel': 1.0, 'eta': 0.05, 'gamma': 0.3,'lambda': 1.0  | StratifiedKFold(shuffle=False)  |  0.5811 +/- 0.1416  |
| 'max_depth': 3, 'subsample': 0.9, 'min_child_weight': 1.2, 'colsample_bytree': 0.2,'colsample_bylevel': 1.0, 'eta': 0.05, 'gamma': 0.3,'lambda': 1.0  | StratifiedKFold(shuffle=True)  |   0.7723 +/- 0.0181  |

#### 多模型融合
**Simple Average**

**Linear Blending**

**k-fold 2-level Stacking**