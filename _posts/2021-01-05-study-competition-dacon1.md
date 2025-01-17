---
layout: post
title:  "[Dacon] AI야, 진짜 뉴스를 찾아줘! AI 경진대회"
date:   '2021-01-05' 
categories: study 
tags : [competition,  dacon, NLP]
comments : true
---


```python
import numpy as np
import pandas as pd
import csv
import os
```

#### 데이터 불러오기 


```python
path='C:/Users/1868j/OneDrive/바탕 화면/리그1/open/'
train = pd.read_csv(path+'news_train.csv') # train.csv 불러오기
test = pd.read_csv(path+'news_test.csv') # test.csv 불러오기
submission=pd.read_csv(path+'sample_submission.csv')

y_train=train.iloc[:,5].values
```

#### 시간 측정 시작


```python
import time
start = time.time()
```

#### Library 불러오기

#### pos_Tagger, Tokenizer, pretraind_embedding, Model 불러오기


```python
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
import re
```

#### 형태소 분석 + 전처리


```python
#train data 불용어 제거 &형태소분리
from konlpy.tag import Mecab
mecab = Mecab(dicpath=r"C:\mecab\mecab-ko-dic")
def text_preprocessing(text_list):
    
    stopwords =['을','를','이','가','은','는','null']
    tokenizer = Mecab(dicpath=r"C:\mecab\mecab-ko-dic")
    token_list = []
    
    for text in text_list:
        txt = re.sub('[^가-힣a-zA-Z0-9]', ' ', text.lower())
        token = tokenizer.morphs(txt)
        token = [t for t in token if t not in stopwords or type(t) !=float]
        token_list.append(token)
        
    return token_list, tokenizer

train['new_article'], Mecab = text_preprocessing(train['content'])
```


```python
train['new_article']
```




    0         [이데일리, marketpoint, 15, 32, 현재, 코스닥, 기관, 678, ...
    1         [실적, 기반, 저가, 에, 매집, 해야, 할, 8, 월, 급등유망주, top, 5...
    2               [하이스탁론, 선취수수료, 없, 는, 월, 0, 4, 최저금리, 상품, 출시]
    3              [종합, 경제, 정보, 미디어, 이데일리, 무, 단전, 재, 재배, 포, 금지]
    4                       [전국, 적, 인, 소비, 붐, 조성, 에, 기여, 할, 예정]
                                    ...                        
    118740    [미, fda, 임상, 3, 상, 허가, 임박, 묻, 고, 따, 블, 로, 갈, 바...
    118741                    [똑똑, 해진, 소비자, 한국, 도, 이젠, 소형차, 시대]
    118742                    [똑똑, 해진, 소비자, 한국, 도, 이젠, 소형차, 시대]
    118743          [2020, 년, 한국, tv, 2, 대중, 1, 대, 인터넷, 연결, 된다]
    118744          [2020, 년, 한국, tv, 2, 대중, 1, 대, 인터넷, 연결, 된다]
    Name: new_article, Length: 118745, dtype: object




```python
from konlpy.tag import Mecab
test['new_article'], Mecab = text_preprocessing(test['content'])
```


```python
tokenizer = Tokenizer(vocab_size, oov_token = 'OOV')  #vocabsize: 13587개! 
tokenizer.fit_on_texts(train['new_article'])
X_train = tokenizer.texts_to_sequences(train['new_article'])
X_test = tokenizer.texts_to_sequences(test['new_article'])
```


```python
y_train = np.array(train['info'])
```


```python
#빈도수가 낮은 단어를 삭제해 생긴 빈 샘플들을 제거 
drop_train = [index for index, sentence in enumerate(X_train) if len(sentence) < 1]
```


```python
# 빈 샘플들을 제거
X_train = np.delete(X_train, drop_train, axis=0)
y_train = np.delete(y_train, drop_train, axis=0)
```

    C:\Users\1868j\AppData\Local\Continuum\anaconda3\lib\site-packages\numpy\core\_asarray.py:83: VisibleDeprecationWarning: Creating an ndarray from ragged nested sequences (which is a list-or-tuple of lists-or-tuples-or ndarrays with different lengths or shapes) is deprecated. If you meant to do this, you must specify 'dtype=object' when creating the ndarray
      return array(a, dtype, copy=False, order=order)



```python
max_len = 75
X_train = pad_sequences(X_train, maxlen = max_len)
X_test = pad_sequences(X_test, maxlen = max_len)
```


```python
from keras.models import Sequential
from keras.layers import Embedding, Flatten, Dense, LSTM, BatchNormalization, Dropout,SpatialDropout1D
from tensorflow.keras import regularizers
model = Sequential()
model.add(Embedding(vocab_size, 100,input_length=100))
model.add(SpatialDropout1D(0.3))
model.add(LSTM(64))
model.add(Dropout(0.5))
model.add(Dense(64, activation='relu', kernel_regularizer = regularizers.l2(0.001)))
model.add(Dense(1, activation='sigmoid'))
model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics='accuracy')
model.summary()
```

    Model: "sequential_1"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_1 (Embedding)      (None, 100, 100)          2392500   
    _________________________________________________________________
    spatial_dropout1d_1 (Spatial (None, 100, 100)          0         
    _________________________________________________________________
    lstm_1 (LSTM)                (None, 64)                42240     
    _________________________________________________________________
    dropout_1 (Dropout)          (None, 64)                0         
    _________________________________________________________________
    dense_2 (Dense)              (None, 64)                4160      
    _________________________________________________________________
    dense_3 (Dense)              (None, 1)                 65        
    =================================================================
    Total params: 2,438,965
    Trainable params: 2,438,965
    Non-trainable params: 0
    _________________________________________________________________



```python
history = model.fit(X_train, y_train,epochs=100,batch_size=128,validation_split=0.2)
```

    Epoch 1/100
    743/743 [==============================] - 114s 153ms/step - loss: 0.0578 - accuracy: 0.9800 - val_loss: 0.0396 - val_accuracy: 0.9830
    Epoch 2/100
    743/743 [==============================] - 114s 154ms/step - loss: 0.0479 - accuracy: 0.9837 - val_loss: 0.0748 - val_accuracy: 0.9670
    ...
    Epoch 99/100
    743/743 [==============================] - 78s 105ms/step - loss: 0.0023 - accuracy: 0.9994 - val_loss: 0.1461 - val_accuracy: 0.9769
    Epoch 100/100
    743/743 [==============================] - 77s 104ms/step - loss: 0.0021 - accuracy: 0.9994 - val_loss: 0.1601 - val_accuracy: 0.9737




#### 예측


```python
submission['info']= model.predict(X_test)
```


```python
submission.loc[(submission['info']>=0.5), 'info'] = 1
submission.loc[(submission['info']<0.5), 'info'] = 0
submission.head()
```


```python
print(time.time()-start)
```

    260.0530252456665


#### 제출


```python
 #submission.to_csv("submission_9(dacon).csv", index=False)
```
