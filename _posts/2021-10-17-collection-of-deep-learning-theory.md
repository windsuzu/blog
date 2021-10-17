---
layout: post
title: 📌 深度學習的理論目錄 - Collection of Deep Learning Theory
description: 
categories: [Deep Learning, Recurrent Neural Network, RNN, Convolutional Neural Network, CNN, Collection]
sticky_rank: 2
---

# Prerequisite

建議看完 [機器學習的理論]({% post_url 2021-10-15-collection-of-machine-learning-theory %}) 並學會機器學習的基礎概念後，再來研究深度學習。

# Introduction

深度學習是由機器學習分支出來的一個大領域，其概念是應用神經網路為基礎，並以現今硬體 (特別是 GPU) 效能為輔助，將神經網路建立的越來越大 (**深度**兩字的由來)。網路內的所有神經元 (neuron) 可以自動學習到問題所需的參數，並以此來預測、解決未知的問題。

吳恩達教授的深度學習 {% fn 1 %} 是我最推薦的深度學習基礎課程之一，這裡一樣是在吸收完教學後，所寫的一些教學筆記；我將所有筆記統整在這裡，方便大家能夠一起學習。

{% include alert.html text="這是新的部落格，目前還在搬運筆記中，舊筆記在: https://sejkai.gitbook.io/academic/deep-learning"%}

# Catalog

## Deep Neural Network (DNN)

- [深度學習基礎理論 - 什麼是神經網路 (Neural Network)]()
- [深度學習基礎理論 - 怎麼訓練神經網路 - 反向傳播算法 (Backpropagation)]()
- [深度學習基礎理論 - 怎麼訓練神經網路 - 激勵函數 (Activation Function)]()
- [深度學習基礎理論 - 淺層的神經網路 (SNN, Shallow Neural Network)]()
- [深度學習基礎理論 - 深層的神經網路 (DNN, Deep Neural Network)]()
- [深度學習基礎理論 - 對神經網路正則化 (Regularization)]()
- [深度學習基礎理論 - Dropout 正則化]()
- [深度學習基礎理論 - 優化神經網路 (Optimization)]()
- [深度學習基礎理論 - Momentum, RMSprop, Adam 優化]()
- [深度學習基礎理論 - 超參數調整 (Hyperparameter Tuning)]()
- [深度學習基礎理論 - 批量歸一化 (BN, Batch Normalization)]()
- [深度學習基礎理論 - Softmax 函式 (Softmax Regression)]()

## Convolutional Neural Network (CNN)

- [深度學習基礎理論 - 計算機視覺與卷積運算 (Computer Vision and Convolutional Operation)]()
- [深度學習基礎理論 - 卷積神經網路 (CNN, Convolutional Neural Network)]()
- [深度學習基礎理論 - 經典的卷積神經網路 (LeNet-5, AlexNet, VGG)]()
- [深度學習基礎理論 - 殘差網路 (ResNets, Residual Networks)]()
- [深度學習基礎理論 - Inception 網路]()
- [深度學習基礎理論 - 物件偵測 (Object Detection)]()
- [深度學習基礎理論 - YOLO, You Only Look Once]()
- [深度學習基礎理論 - R-CNN]()
- [深度學習基礎理論 - 人臉識別 (Face Recognition)]()
- [深度學習基礎理論 - 風格轉換 (Neural Style Transfer)]()

## Recurrent Neural Network (RNN)

- [深度學習基礎理論 - 循環神經網路 (RNN, Recurrent Neural Network)]()
- [深度學習基礎理論 - 語言模型 (Language Model)]()
- [深度學習基礎理論 - GRU 和 LSTM (Gated Recurrent Unit & Long Short Term Memory)]()
- [深度學習基礎理論 - 雙向、深度循環神經網路 (Bidirectional RNN & Deep RNN)]()
- [深度學習基礎理論 - 詞嵌入 (Word Embedding)]()
- [深度學習基礎理論 - Word2Vec 詞嵌入]()
- [深度學習基礎理論 - 情感分類 (Sentiment Classification)]()
- [深度學習基礎理論 - 序列模型 (Seq2Seq, Sequence-to-Sequence Model)]()
- [深度學習基礎理論 - 光束搜尋 (Beam Search)]()
- [深度學習基礎理論 - BLEU (Bilingual Evaluation Understudy)]()
- [深度學習基礎理論 - 注意力模型 (Attention Model)]()

# Credit

- Teached by [Andrew Ng](https://www.coursera.org/instructor/andrewng), CEO/Founder Landing AI; Co-founder, Coursera; Adjunct Professor, Stanford University; formerly Chief Scientist,Baidu and founding lead of Google Brain
- Created by [Stanford University](https://www.coursera.org/learn/machine-learning)


{{ "https://zh-tw.coursera.org/specializations/deep-learning" | fndetail: 1 }}
