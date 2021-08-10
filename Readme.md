# Kaggle SIIM-FISABIO-RSNA COVID-19 Detection 15th   Solution

## 比赛介绍
1. 参赛者通过构建计算机视觉模型通过X-ray胸片诊断患者是否患有COVID-19，并且对肺部感染区域进行定位。
2. 这次比赛是 图像分类 + 目标检测 两种模型的融合，来获得最终结果,可以尝试EffecientNet + YOLOv5的思路来完成。


## 比赛数据
1. 赛题数据非常大（约100G），但有只有6000多条的训练数据,原因是数据采用的医疗影像的专用格式DCM，每张图片占空间较大。
2. 简单预览数据后，我们会发现，数据分为两个层级，study_level 和 image_level。
Study代表一次检查，image代表一张图片。一次检查可能会包含多张图片（原因可能是拍片位置不好、成像不清楚等等）。训练集一共包含6054次检查和6334个图片。
3. 图像分类是针对study_level的，一个study只可能包含以下四种结果中的一种： “阴性”, “典型”, “不确定”, “非典型”
目标检测是针对image_level的，只有一种目标： opacity（肺部不透明度），但一张图片可以有多处opacity。


## 最终方案
1. 用EfficientnetV2-L-in21k做4分类任务，完成5个Fold的训练后，可以得到纯分类的PublicLB 0.462。
2. 用修改过结构的Yolov5做目标检测，完成5个Fold的训练后，可以将结果提升PublicLB 0.636。
3. 用WBF融合刚才的Yolov5和公开的CascadeRCNN（ https://www.kaggle.com/sreevishnudamodaran/siim-effnetv2-l-cascadercnn-mmdetection-infer ），可以将结果提升PublicLB 0.638。
4. 对于目标检测中的none值，用EfficientnetV2-L-in21k做一个2分类任务（同样5Fold）来预测，可以将结果提升PublicLB 0.644。


## 比赛Trick点
1. 比赛数据样本不均衡，在分类中使用FocalLoss，涨分0.02左右。
2. 数据较少，为了防止过拟合，使用aux task增加训练任务的难度，涨分0.05左右。（ https://www.kaggle.com/c/siim-covid19-detection/discussion/240233 ）
3. 这次比赛对超参非常敏感， lr loss aug 都很敏感，容易过拟合，使用optuna调参，涨分0.05。
4. WBF做目标检测融合效果很好，涨分0.02。
5. 在推理阶段，图像分类使用hflip做tta，涨分0.01~0.02。



## 尝试后，没有效果的
1. 因为数据集数量少，于是引入外部数据集RSNA数据做softlabel，但没有涨分。
2. 在Yolov5中引入aux task，代码很复杂，可能是因为代码逻辑的问题，没有涨分，但后面也没有再花时间。

