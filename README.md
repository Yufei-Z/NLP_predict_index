# 中文NLP预测股价变化

**Goal**:  trying to use NLP tech to predict the stock. 

## literature review

[S&P market intelligence](http://pages.marketintelligence.spglobal.com/rs/565-BDO-100/images/MI-Research-NLPLitSurvey-170725.pdf?mkt_tok=eyJpIjoiWlRoa016WmlZVEZpT1RRMyIsInQiOiJ2bklHRUptZFwvMFlDQ3duK3c3VGRPbklqMEpZM3dJVlhEb29GWng0bnlHRVFMbWVBdUlLV1VUQ2R4dW4xaExIYlRkRkVvbXBNT0tHRmFyRHY5V0R1a3VxZUNybkRzYjd5eXNPVzh0bVFLOEhhTndTTzJOY2JrTm5LY2NIWFlwXC9qIn0%3D) mainly foucus on earning call transcript analysis, which can be a direction for further improvement.


## Method 1: train our own model
Existing baseline model

[股市情感分析（股吧）](https://github.com/algosenses/Stock_Market_Sentiment_Analysis)先把文本tokenlize，然后用标注好的中文文本训练（with lab postive or nagative），然后用各种模型训练，最后得到sentiment score.使用2018年数据


>Remark:

 1.Although the orignial author did segmamention in raw dataset, at the tokenization step his model still divide a sentence into chars instead of words.
 2. Although from above picture it fits well, when we foucs on clsoe price change, you can see its prediction power is not so appealing...
 ![image](https://user-images.githubusercontent.com/39251819/75966522-758ef600-5f05-11ea-94bc-1de95708e369.png)
 
 I tried this model and use it to predict the stock '000002.SZ'. Similar as above, results are frustrating😒. Worse still, my correlation is much lower. The reason may be that the training set is too small and not suitable.( the training set is from the forums of index, not of this stock) 
 
 
 

### Reflection: is word segamentation necessary?
In Chinese, a word, instead of a character, is the smallest unit to express meaning. However, in reality using char is a better idea.
![image](https://user-images.githubusercontent.com/39251819/75848313-cb3ba380-5e1c-11ea-9e95-0c66b8916911.png)
There are several reasons why it's unnecessary: 
 1. segamentation in Chinese is much more difficult to implement，and there are different standards for word segamentation. This process will introduce another kind of error. 
 2. we don't know in neutral network' eyes, whether words or characters express the meanings better. With the development of deep learning models，like RNN, LSTM, BERT, models using charaters improved a lot. 
 3. the word matrix is too sparse, meaning high overfitting risk. And there are lot's of OOV, which restricts the model's preformance.
 4. to be continoued
 
## Method 2: transfer learning
use ernie model provided by baidu

I tried: 
 1. use dataset provided by baidu and may financial dataset.
 2. use my financial dataset only.

The first method achieved 90% accuracy in testset. The I use this model to classify sentiment socre and calculate each day's sentiment score. With the sentiment score we can start to construct our strategy. Here I set the first half data as train set and second half as test set. The normal strategy is relatively trival here: long if the sentiment score larger than mean plus 0.5 sd and sell if smaller mean minus 0.5 sd, after i+1 days. Here i is the look back window, a non negative integer that I want to optimize in train set.

Below is part of the result:

1.600519.SH Moutai
![image](https://user-images.githubusercontent.com/39251819/80866833-8a002c80-8cc3-11ea-9d5e-05ea5237e7b1.png)
![image](https://user-images.githubusercontent.com/39251819/80867009-72757380-8cc4-11ea-8c1f-c150a07568c2.png)

2.000063.SZ ZTE

 i=2  
![image](https://user-images.githubusercontent.com/39251819/80867095-dc8e1880-8cc4-11ea-9a37-6cb706e8e965.png)

![image](https://user-images.githubusercontent.com/39251819/80867112-f3346f80-8cc4-11ea-84aa-870a98b87b00.png)
<p align="center">annual return 126% & sharpe 2.3<p align="center">
 
i=3  
![image](https://user-images.githubusercontent.com/39251819/80867146-237c0e00-8cc5-11ea-9726-87ac00bdf8e2.png)
![image](https://user-images.githubusercontent.com/39251819/80867443-95088c00-8cc6-11ea-8260-26071d05049b.png) 
<p align="center">annual return 40% & sharpe 1.8<p align="center">
 
3. 000858.SH

![image](https://user-images.githubusercontent.com/39251819/80867590-840c4a80-8cc7-11ea-9ce2-692314baac7a.png)

![image](https://user-images.githubusercontent.com/39251819/80867607-97b7b100-8cc7-11ea-9efd-9a627607b12d.png)

<p align="center">Remark: low performance.  <p align="center">

>Remark

 1.The model in fact returns two sentiment score: one discrete label [0,1,2] and one continuous positive probability. The results are similar.
 
 2.The strategy for 000858 seems to doesn't work at all. Reason may be the market has changed a lot(for example, before and after 2015). Strategy should be adjusted every one or two year
 
 3.Sometimes the stock price go to the opposite direction of market sentiment. Based on my limited knowledge, some possible explanation: 
   - There are lots of informed traders in A markets, therefore if there is some good news, the informed traders would long stock and  stock price will increased before the chaos traders read the good news, then the informed traders will try to sell their stocks and price went down.
   - People discard opposite emotion in forums on purpose (to mislead other traders)
   -if all participants have long position, no more free cash to buy, when ppl unwind, it triggers others to cut loss or unwinding position in a row, therefore, a stronger momentum
 
 4 follow 3, I found in liquor industry(wu liangye, moutai) the normal strategy will make profit, where for the rest stocks I have tried 
it is the contrarian strategy that wins. So may it also depends on the industry? To be verified by investigating more stocks...

## work list...
 
- [x] tokenzie before word2vec
- [x] study the relationship between sentiment score in weekends and the pct_change in next Monday
- [x] analyze the earning call scripts, making use of knowledge in behaviour finance.
 
 
 ## relevent kaggle competition
 [predict disaster](https://www.kaggle.com/c/nlp-getting-started/overview)
 keep track of the new kernel and discussion part
 
 ## **Appendix**
 ### previous studies which may be helpful
simply use LSTM to predict(like AR model)
: surprisingly, result is good! corr = 0.82
![image](https://user-images.githubusercontent.com/39251819/73824900-f212b400-4835-11ea-969e-6a1700ec8d1f.png)

[Sentiment Analysis for Event-Driven Stock Prediction](https://github.com/WayneDW/Sentiment-Analysis-in-Event-Driven-Stock-Price-Movement-Prediction#sentiment-analysis-for-event-driven-stock-prediction): Train a set of Bayesian Convolutional Neural Networks using Stochastic Gradient Langevin Dynamics to obtain more robustness

[lastest news classficaiton](https://github.com/miguelfzafra/Latest-News-Classifier) its code can be modified to scratch news online.
### Reference
Is Word Segmentation Necessary for Deep Learning of Chinese Representations?
author：Yuxian Meng, Xiaoya Li, Xiaofei Sun, Qinghong Han, Arianna Yuan, and Jiwei Li
