# 基于多模态检索的互联网图文匹配
## 运行环境
Google Colaboratory  
LLaVa部分需要A100-40G，其余16G即可
## 参考
原始CLIP：https://github.com/openai/CLIP  
中文CLIP：https://huggingface.co/IDEA-CCNL/Taiyi-CLIP-Roberta-large-326M-Chinese  
LLaVA：https://github.com/haotian-liu/LLaVA  
CLIP-Adapter(Tip-Adapter)：https://github.com/gaopengcuhk/Tip-Adapter  
## 项目背景
图文检索是当前最重要的多模态能力之一，针对互联网或者AI公司积累的大量历史数据，以及手机图库中的图片数据，利用多模态大模型对这些数据进行检索，是数据治理的关键。
## 项目内容
### 1. 数据集：
- 数据来源：爬取多类别的互联网图像数据2000张
- 样本选择：选择5个正样本，每个正样本对应易混淆的负样本，5个正样本中有2个和中国特色相关的词语描述。每个正负样本200张图，10个正负样本共2000张图片。
### 2. 项目流程：
中英文CLIP图文匹配+多模态大模型LLaVA结果矫正
1. 先用英文CLIP模型对图片以及文本标签进行匹配。  
2. 设置对比实验：
- 同时输入5个类别以计算相似度的方式来计算precision、recall和F1 score
- 只用其中一个类别+其他类计算precision和recall和F1 Score
- 通过卡阈值的方式，只输入一个类别计算precision和recall和F1 Score  
在上面三种方式中选择精度最高的。
3. 使用中文CLIP模型，按照精度最高的方式进行图文匹配，并计算精度。  
原因：数据集中加入了中国特色的词语，中文模型在中文领域有更好的匹配效果。
4. 将英文和中文CLIP的匹配结果取并集，去重后作为正样本，并计算精度。  
原因：中英文模型取并集的结果可以尽可能召回更多的正样本，减少遗漏，提升recall。  
5. 中英文CLIP的结果经过LLaVA做矫正，保留LLaVA输出正确的正样本，并计算精度。  
原因：合并后召回的图片数量增加，precision不够高，LLaVA可以进一步筛选，提高precision。
6. 额外收集5个正样本的1000张图使用CLIP-Adapter进行微调训练，在之前的2000张图上做测试，然后从1000张图中抽取10，100，500张图片用于训练的测试对比。
## 项目结论
1. 对于原始英文CLIP，精度最高的方式为卡阈值，随着阈值的增加，precision会逐渐提高，recall则逐渐降低，当阈值为0.24时，整体精度最高：Average Precision: 0.7740, Average Recall: 0.8800, Average F1: 0.7984。
2. 合并中英文CLIP的匹配结果，Average Precision: 0.7794, Average Recall: 0.9830, Average F1: 0.8519，不损失precison的情况下，将recall从88%提高到了98.3%，提高了11.7%。
3. 经过LLaVA矫正，Average Precision: 0.9466, Average Recall: 0.9720, Average F1: 0.9575，损失少量recall，提高了21.5%precison，相比于原始CLIP，最终精度(F1 Score)提高了19.9%。
4. 经过CLIP-Adapter的微调，100张图片的训练几乎没有提高精度，500张图片的训练将精度提高了2.37%，1000张图片的训练将精度提高了2.93%。
## 不足与展望
1. 数据集使用了中文互联网的图片数据，中文CLIP已经有很好的效果，可以尝试不同的数据来源进行测试。
2. 使用LLaVA对结果进行矫正有不错的效果，不过precision还有提升的空间，可以在特定的VQA场景上进行微调。
3. CLIP-Adapter进行微调时使用的数据量较小，效果不太明显，可以尝试构建更大量级的微调数据集。
