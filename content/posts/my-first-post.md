+++
date = '2026-09-16T17:59:55+08:00'
draft = false
title = 'My First Post'

+++



这是我第一篇关于AI的博客

ghout.</think># 关于扩展 MedicalAiSystem 项目的建议

先说明一点：我无法实时访问 GitHub 仓库，以下建议基于此类医疗 AI 项目的典型结构（Flask + sklearn 模型 + HTML 前端）。你可以对照实际代码确认，如果项目结构差异较大，可以把目录结构发给我，我再针对性分析。

## 一、先读懂项目的核心流程

这类项目通常遵循这条链路，添加功能前建议先梳理清楚：

text



前端表单 → 路由接收数据 → 加载模型 → 预测 → 返回结果渲染页面

重点关注三个位置：

- `app.py` 中的路由函数（数据怎么进来）
- 模型加载和预测的代码（`joblib.load()` / `pickle.load()`）
- `templates/` 下的表单页面（数据格式和字段）

## 二、如何添加自己的功能（实操示例）

以**添加一种新的疾病预测**为例，这是最典型的扩展方式：

**第 1 步：训练新模型**

python



\# train_new_model.py

import pandas as pd

from sklearn.ensemble import RandomForestClassifier

import joblib

data = pd.read_csv('data/liver_disease.csv')  # 从 UCI/Kaggle 下载

X = data.drop('target', axis=1)

y = data['target']

model = RandomForestClassifier()

model.fit(X, y)

joblib.dump(model, 'model/liver_model.pkl')

**第 2 步：在 app.py 添加路由**（模仿现有预测路由的写法）

python



@app.route('/liver_predict', methods=['POST'])

def liver_predict():

​    features = [float(request.form[k]) for k in ['age', 'bilirubin', 'albumin', ...]]

​    model = joblib.load('model/liver_model.pkl')

​    result = model.predict([features])[0]

​    proba = model.predict_proba([features])[0]  # 顺便返回概率

​    return render_template('liver_result.html', result=result, proba=proba)

**第 3 步：添加前端页面**，复制现有表单页面改字段即可。

这个套路掌握后，本质上你可以无限扩展“新疾病预测模块”。

## 三、可以扩展的功能方向（按难度分级）

**⭐ 入门级（1-2天）**

- 接入更多疾病数据集（UCI 有心脏病、糖尿病、肝病、乳腺癌等经典数据集）
- 预测结果可视化：用 ECharts 画概率条形图、雷达图展示各项指标
- 历史预测记录（存到 SQLite，展示查询历史）
- 前端美化（Bootstrap / Element UI）

**⭐⭐ 进阶级（适合课程设计/毕设加分）**

- **用户系统**：Flask-Login 实现注册登录，用户各自管理健康档案
- **症状自查**：用户勾选多个症状 → 用朴素贝叶斯/规则匹配推荐可能疾病
- **多模型对比**：同一疾病跑 SVM、随机森林、XGBoost，展示各模型准确率对比
- **药品与科室推荐**：根据诊断结果推荐挂号科室、常用药说明
- **模型可解释性**：集成 SHAP，展示哪些指标影响了预测结果（医疗场景很加分）

**⭐⭐⭐ 高阶级（可以往简历/竞赛方向做）**

- **医学影像识别**：用 CNN（PyTorch）做 X光片肺炎识别、皮肤病变分类，Pneumonia-MNIST 等数据集容易上手
- **智能问答机器人**：接入大模型 API（如通义千问/DeepSeek），做医疗咨询对话，配合 RAG 检索医学知识库
- **电子病历 NLP 分析**：命名实体识别提取病历中的症状、药物实体
- **前后端分离重构**：Vue/React 前端 + Flask/FastAPI 后端 + 接口文档

## 四、几个建议

1. **循序渐进**：先做入门级功能熟悉全流程，再挑一个进阶功能做深，比铺开一堆浅功能更有价值
2. **加免责声明**：医疗 AI 项目要注明“仅供参考，不构成医疗建议”，这是基本的合规意识
3. **保留 git 记录**：每加一个功能 commit 一次，既方便回滚也能体现你的开发过程
4. **挑一个有亮点的方向**：如果是为了求职或毕设，“影像识别”和“LLM 问答 + RAG”是目前最有吸引力的两个方向

你目前的项目具体包含哪些功能（比如已经支持哪些疾病预测）？用的什么技术栈？告诉我后我可以给出更具体的改造方案。我可以给出更具体的改造方案。
