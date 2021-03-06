## PPD_Overdue_Prediction 拍拍贷贷款逾期概率预测
### 描述 & 数据 & 目标
- 描述
    - [“魔镜杯”风控算法大赛](https://www.kesci.com/apps/home/competition/56cd5f02b89b5bd026cb39c9/content)
    - 原赛题共两轮，初赛轮提供3万条训练集数据，2万条测试集数据；复赛轮除提供初赛轮数据外，新增3万条训练集数据，1万条测试集数据。
    - 本repo目前仅使用初赛轮数据。
- 数据
    - 数据包括违约标签（因变量）、建模所需的基础与加工字段（自变量）、相关用户的网络行为原始数据；*数据字段已经过脱敏处理*。包括三种数据：
        - Master：每一行代表一个样本（一笔成功成交借款），每个样本包含200多个各类字段。
            - idx：每一笔贷款的unique key，可以与另外2个文件里的idx相匹配。
            - UserInfo_*：借款人特征字段
            - WeblogInfo_*：Info网络行为字段
            - Education_Info*：学历学籍字段
            - ThirdParty_Info_PeriodN_*：第三方数据时间段N字段
            - SocialNetwork_*：社交网络字段
            - ListingInfo：借款成交时间
            - Target：违约标签（1 = 贷款违约，0 = 正常还款）。测试集里不包含target字段。
        - Log_Info：借款人登陆日志。
            - ListingInfo：借款成交时间
            - LogInfo1：操作代码
            - LogInfo2：操作类别
            - LogInfo3：登陆时间
            - idx：每一笔贷款的unique key
        - Userupdate_Info：借款人信息修改日志。
            - ListingInfo1：借款成交时间
            - UserupdateInfo1：修改内容
            - UserupdateInfo2：修改时间
            - idx：每一笔贷款的unique key
- 目标
    - 预测测试集每个样本的违约概率；评价指标为AUC。