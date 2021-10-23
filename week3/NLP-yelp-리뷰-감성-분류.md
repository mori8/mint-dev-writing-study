> 이 글은 한빛미디어의 ['파이토치로 배우는 자연어처리'](http://www.yes24.com/Product/Goods/101874047) 글을 읽고 정리한 것입니다.

퍼셉트론과 지도 학습 훈련 방법을 사용해 [옐프(Yelp)](https://ko.wikipedia.org/wiki/%EC%98%90%ED%94%84) 의 레스토랑 리뷰가 긍정적인지 부정적인지 분류하는 작업을 해보자. 데이터를 로드하고 전처리하는 전체 코드는 [여기](https://github.com/rickiepark/nlp-with-pytorch/blob/main/chapter_3/3_5_yelp_dataset_preprocessing_LITE.ipynb)서 확인할 수 있다.

### Dataset: 옐프 리뷰 데이터셋

이 실습에서는 2015년에 옐프가 주최한 레스토랑 등급 예측 대회에서 사용되었던 레스토랑 리뷰 데이터를 간소화한 뒤 훈련 샘플과 테스트 샘플로 분류해 둔 데이터셋 중 10%를 사용한다.

전체 데이터셋 중 일부만 사용하는 이유는 크게 2가지가 있다.

1.  작은 데이터셋을 사용하면 훈련-테스트 반복이 빠르게 이루어져 **실험 속도를 높일 수 있다.**
2.  전체 데이터를 사용할 때보다 모델의 정확도가 낮다. 낮은 정확도는 일반적으로 크게 문제가 되지 않으며, **작은 데이터셋에서 얻은 지식으로 전테 데이터셋에서 모델을 다시 훈련할 수 있기 때문에**, 훈련 데이터 양이 아주 많을 때 딥러닝 모델을 훈련할 수 있는 유용한 방법이다.

### Preprocessing: 데이터 가져오고 다듬기

#### 데이터 가져오기

우선 데이터 전처리에 필요한 라이브러리를 임포트하자. 천리길도 임포트부터 🧗🏻‍♀️

```
import numpy as np
import pandas as pd
import regex as re
import collections

from argparse import Namespace
```

전처리 과정에서 사용할 변수들도 세팅하자.

```
args = Namespace(
    raw_train_dataset_csv="/content/drive/MyDrive/NLP-Pytorch/data/yelp/raw_train.csv",
    raw_test_dataset_csv="/content/drive/MyDrive/NLP-Pytorch/data/yelp/raw_test.csv",
    proportion_subset_of_train=0.1,
    train_proportion=0.7,
    val_proportion=0.15,
    test_proportion=0.15,
    output_munged_csv="/content/drive/MyDrive/NLP-Pytorch/data/yelp/reviews_with_splits_lite.csv",
    seed=1337
)
```

이제 본격적으로 데이터를 다뤄보자. 먼저, 원본 데이터를 가져온 다음 `head()`로 데이터의 생김새를 간단하게 파악하자.

```
train_reviews = pd.read_csv(args.raw_train_dataset_csv, header=None, names=['rating', 'review'])
train_reviews.head()
```

|   | rating | review |
| --- | --- | --- |
| 0 | 1 | Unfortunately, the frustration of being Dr.Go... |
| 1 | 2 | Been going to Dr. Goldberg for over 10 years. ... |
| 2 | 1 | I don't know what Dr. Goldberg was like before... |
| 3 | 1 | I'm writing this review to give you a heads up... |
| 4 | 2 | All the foods is great here. But the best thing... |

#### 데이터 분리하기

우리는 전체 데이터셋 중 10%에 해당하는 서브셋만 사용할 것이다. 하지만 이 서브셋에 특정 rating 값을 가진 데이터만 편중되게 포함된다면 학습이 제대로 진행되지 않을 것이다. 그래서 **서브셋 내의 각 rating 클래스 비율이 동일**하도록, 표본집단이 모집단의 성질을 반영하도록 서브셋을 구성해야 한다.

```
by_rating = collections.defaultdict(list)
for _, row in train_reviews.iterrows():
  by_rating[row.rating].append(row.to_dict())

review_subset = []

for _, item_list in sorted(by_rating.items()):
  n_total = len(item_list)  # 전체 데이터셋 중 해당 rating value를 갖는 데이터의 개수
  n_subset = int(args.proportion_subset_of_train * n_total)
  review_subset.extend(item_list[:n_subset])  # 전체 데이터셋 중 n_total * 0.1만큼만 슬라이싱하여 사용

review_subset = pd.DataFrame(review_subset)
review_subset.head()
```

|   | rating | review |
| --- | --- | --- |
| 0 | 1 | Unfortunately, the frustration of being Dr.Go... |
| 1 | 1 | I don't know what Dr. Goldberg was like before... |
| 2 | 1 | I'm writing this review to give you a heads up... |
| 3 | 1 | Wing sauce is like water. Pretty much a lot of... |
| 4 | 1 | Owning a driving range inside the city limits ... |

여기서 데이터셋의 각 rating 클래스의 개수를 세어보면 서로 동일한 개수의 데이터를 갖는다.

```
train_reviews.rating.value_counts()    # 2 280000 / 1 280000
```

```
set(review_subset.rating)    # {1, 2}
```

이렇게 처리된 데이터셋을 훈련, 검증, 테스트 세트로 분리해야 한다. 미리 정해둔 각 세트의 비율에 따라 데이터를 나눈다. 각 세트의 비율은 이 글의 2번째 코드블럭에서 선언한 `args` Namespace에 정의되어 있으며, `args.train_proportion` = 0.75, `args.val_proportion` = 0.15, `args.test_proportion` = 0.15다. 3개의 세트로 분리한 뒤에는 `split` 속성에 이 데이터가 어느 세트에 속한 데이터인지 표시한다.

```
# 별점 기준으로 나누어 훈련, 검증, 테스트 데이터셋을 만든다.
by_rating = collections.defaultdict(list)
for _, row in review_subset.iterrows():
  by_rating[row.rating].append(row.to_dict())

# 분할 데이터를 만든다.
final_list = []
np.random.seed(args.seed)

for _, item_list in sorted(by_rating.items()):
  np.random.shuffle(item_list)
  n_total = len(item_list)
  n_train = int(args.train_proportion * n_total)
  n_val = int(args.val_proportion * n_total)
  n_test = int(args.test_proportion * n_total)
    # 각 데이터에 레이블을 붙여준다.
  for item in item_list[:n_train]:
    item['split'] = 'train'

  for item in item_list[n_train:n_train+n_val]:
    item['split'] = 'val'

  for item in item_list[n_train+n_val:n_train+n_val+n_test]:
    item['split'] = 'test'

  final_list.extend(item_list)

final_reviews = pd.DataFrame(final_list)
```

#### 데이터 정제하기

이후, 데이터를 정제하는 작업이 필요하다. 우리가 구현할 클래스는 리뷰 데이터를 공백을 기준으로 나누어 토큰을 획득하므로, 구두점 기호 앞뒤에 공백을 넣고, 구두점이 아닌 다른 기호들은 제거한다.

```
def preprocess_text(text):
  text = text.lower()
  text = re.sub(r"([.,!?])", r" \1 ", text)
  text = re.sub(r"[^a-zA-Z.,!?]+", r" ", text)

  return text

final_reviews.review = final_reviews.review.apply(preprocess_text)
```

다음으로, 숫자(1, 2)로 표시된 `rating` 열의 값들을 각각 'negative'와 'positive'로 바꿔준다.

```
final_reviews['rating'] = final_reviews.rating.apply({1: 'negative', 2: 'positive'}.get)
```

잘 바뀌었는지 `final_reviews.head()`로 확인해 보자.

|   | rating | review | split |
| --- | --- | --- | --- |
| 0 | negative | terrible place to work for i just heard a stor... | train |
| 1 | negative | hours , minutes total time for an extremely s... | train |
| 2 | negative | my less then stellar review is for service . w... | train |
| 3 | negative | i m granting one star because there s no way t... | train |
| 4 | negative | the food here is mediocre at best . i went aft... | train |

잘 처리되었다! 이대로 csv로 저장하자 😚

```
final_reviews.to_csv(args.output_munged_csv, index=False)
```

다음으로, 전처리된 데이터셋을 가지고 본격적으로 분류 모델을 만들어볼 것이다.
전처리가 끝난 텍스트 데이터를 토큰화, 벡터화한 후 Dataset으로 만드는 과정을 코드로 작성해보자. 우리는 여기서 `ReviewDataset`, `Vocabulary`, `ReviewVectorizer` 클래스를 만들 것이다.
- `ReviewDataset`: csv 파일을 받아 데이터셋을 로드하고, 이 데이터셋을 바탕으로 `ReviewVectorizer` 객체를 만든다.
- `ReviewVectorizer`: 각각 리뷰와 별점 정보를 담고 있는 `Vocabulary` 객체 2개를 만들어 관리한다.
- `Vocabulary`: 객체는 매핑을 위해 텍스트를 처리하고 어휘 사전을 만드는 클래스로, 각 토큰과 고유값(정수)를 매핑한다.



### 파이토치 데이터셋 이해하기

파이토치는 `Dataset` 클래스로 데이터셋을 추상화한다. `Dataset` 는 추상화된 반복자(iterator) 클래스로, 파이토치에서 새로운 데이터셋을 사용할 때 먼저 `Dataset` 클래스를 상속해서 `__getitem__()`과 `__len__()` 메서드를 필수적으로 구현해야 한다.
아래의 코드에서는 `Dataset`을 상속한 `ReviewDataset` 클래스와 함께 `ReviewVectorizer`와 `DataLoader`를 사용한다.
여기서 구현할 `ReviewDataset` 클래스는 아래 3가지 조건을 만족한다고 가정한다.

- 데이터셋이 최소한으로 정제되어 있고, 훈련, 검증, 테스트 set으로 나누어져 있다.
- 이 데이터셋의 리뷰를 공백을 기준으로 나누면 토큰 리스트를 얻을 수 있다.
- 데이터 샘플에 이 샘플이 훈련, 검증, 세트 중 어느 세트에 포함되어 있는지 표시되어 있다.

아래는 `ReviewDataset`을 구현한 코드다. @classmethod로 클래스의 진입점을 지정했다. @classmethod에 대한 자세한 설명은 [여기](https://wikidocs.net/16074)를 참고하자.
```python
class ReviewDataset(Dataset):
  def __init__(self, review_df, vectorizer):
    """
    매개변수:
      review_df(pandas.DataFrame): 데이터셋
      vectorizer(ReviewVectorizer): ReviewVectorizer 객체
    """
    self.review_df = review_df
    self._vectorizer = vectorizer
    
    self.train_df = self.review_df[self.review_df.split=='train']
    self.train_size = len(self.train_df)

    self.val_df = self.review_df[self.review_df.split=='val']
    self.validation_size = len(self.val_df)

    self.test_df = self.review_df[self.review_df.split=='test']
    self.test_size = len(self.test_df)

    self._lookup_dict = {'train': (self.train_df, self.train_size),
                         'val': (self.val_df, self.validation_size),
                         'test': (self.test_df, self.test_size)}
    
    self.set_split('train')
  
  @classmethod # 정적 메서드, 클래스에서 직접 접근할 수 있음. 자식 클래스인 경우 부모 클래스가 아닌 자식 클래스(자신)의 속성을 사용함
  def load_dataset_and_make_vectorizer(cls, review_csv):
    """
    데이터셋을 로드하고 새로운 ReviewVectorizer 객체를 만든다.
    매개변수:
      review_csv(str): 데이터셋의 위치
    반환값:
      ReviewDataset의 인스턴스
    """
    review_df = pd.read_csv(review_csv)
    train_review_df = review_df[review_df.split=='train']
    return cls(review_df, ReviewVectorizer.from_dataframe(train_review_df))
  
  @classmethod
  def load_dataset_and_load_vectorizer(cls, review_csv, vectorizer_filepath):
    """
    데이터셋을 로드하고 새로운 ReviewVectorizer 객체를 만든다.
    캐시된 ReviewVectorizer 객체를 재사용할 때 사용한다.
    매개변수:
      review_csv(str): 데이터셋의 위치
      vectorizer_filepath(str): ReviewVectorizer 객체의 저장 위치
    반환값:
      ReviewDataset의 인스턴스
    """
    review_df = pd.read_csv(review_csv)
    vectorizer = cls.load_vectorizer_only(vectorizer_filepath)
    return cls(review_df, vectorizer)

  @staticmethod
  def load_vectorizer_only(vectorizer_filepath):
    """
    파일에서 ReviewVectorizer 객체를 로드하는 정적 메서드
    매개변수:
      vectorizer_filepath(str): 직렬화된 ReviewVectorizer 객체의 위치
    반환값:
      ReviewVectorizer의 인스턴스
    """
    with open(vectorizer_filepath) as fp:
      return ReviewVectorizer.from_serializable(json.load(fp))
    
  def save_vectorizer(self, vectorizer_filepath):
    """
    ReviewVectorizer 객체를 json 형태로 디스크에 저장
    매개변수:
      vectorizer_filepath(str): 직렬화된 ReviewVectorizer 객체를 저장할 위치
    """
    with open(vectorizer_filepath, "w") as fp:
      json.dump(self._vectorizer.to_serializable(), fp)
  
  def get_vectorizer(self):
    """ 벡터 변환 객체를 반환 """
    return self._vectorizer
  
  def set_split(self, split="train"):
    """
    데이터프레임에 있는 열을 사용해 데이터셋을 선택
    매개변수:
      split(str): "train" or "val" or "test"
    """
    self._target_split = split
    self._target_df, self._target_size = self._lookup_dict[split]
  
  def __len__(self):
    return self._target_size
  
  def __getitem__(self, index):
    """
    파이토치 데이터셋의 주요 진입 메서드
    매개변수:
      index(int): 데이터 포인트의 인덱스
    반환값:
      데이터 포인트의 특성(x_data)과 레이블(y_target)로 이루어진 딕셔너리
    """
    row = self._target_df.iloc[index]
    review_vector = self._vectorizer.vectorize(row.review)
    rating_index = self._vectorizer.rating_vocab.lookup_token(row.rating)
    return {'x_data': review_vector,
            'y_target': rating_index}
  
  def get_num_batches(self, batch_size):
    """
    배치 크기가 주어지면 데이터셋으로 만들 수 있는 배치 개수를 반환
    매개변수:
      batch_size(int)
    반환값:
      배치 개수
    """
    return len(self) // batch_size
```

### Vocabulary와 ReviewVectorizer, DataLoader
이 예제에서는 `Vocabulary`와 `ReviewVectorizer`, `DataLoader`를 통해 각 토큰(단어)를 정수에 매핑하고, 이 매핑을 각 데이터 포인트에 적용해 벡터 형태로 변환한 뒤에 모델에서 사용하기 위한 미니배치로 모은다.

#### Vocabulary
텍스트를 벡터의 미니배치로 바꾸는 첫번째 단계는 토큰을 정수로 매핑하는 것이다. `Vocabulary` 클래스는 {인덱스:토큰} 딕셔너리와 {토큰:인덱스} 딕셔너리를 캡슐화한다. 사용자가 딕셔너리에 새로운 토큰을 추가하면 자동으로 인덱스를 증가시키고, 효율적인 메모리 관리를 위해 훈련 과정에서 본 적이 없거나 출현 빈도가 낮았던 토큰을 받았을 때에 `UNK`(unknown)라는 특별 토큰으로 처리한다. 
```python
class Vocabulary(object):
  """ 매핑을 위해 텍스트를 처리하고 어휘 사전을 만드는 클래스 """
  def __init__(self, token_to_idx=None, add_unk=True, unk_token="<UNK>"):
    """
    매개변수:
      token_to_idx(dict): 기존 토큰-인덱스 매핑 딕셔너리
      add_unk(bool): UNK 토큰을 추가할지 지정하는 플래그
      unk_token(str): Vocabulary에 추가할 UNK 토큰
    """
    if token_to_idx is None:
      token_to_idx = {}
    self._token_to_idx = token_to_idx
    self._idx_to_token = {idx: token for token, idx in self._token_to_idx.items()}
    self._add_unk = add_unk
    self._unk_token = unk_token
    self.unk_index = -1
    if add_unk:
      self.unk_index = self.add_token(unk_token)
    
  
  def to_serializable(self):
    """ 직렬화할 수 있는 딕셔너리를 반환합니다 """
    return {'token_to_idx': self._token_to_idx,
            'add_unk': self._add_ink,
            'unk_token': self._unk_token}
  
  
  @classmethod
  def from_serializable(cls, contents):
    """ 직렬화된 딕셔너리에서 Vocabulary 객체를 만듭니다 """
    return cls(**contents)  # --> ?
  
  
  def add_token(self, token):
    """ 토큰을 기반으로 매핑 딕셔너리를 업데이트합니다(추가)
    매개변수:
      token(str): Vocabulary에 추가할 토큰
    반환값:
      index(int): 토큰에 상응하는 정수
    """
    if token in self._token_to_idx:
      index = self._token_to_idx[token]
    else:
      index = len(self._token_to_idx)
      self._token_to_idx[token] = index
      self._idx_to_token[index] = token
    return index


    def add_many(self, tokens):
      """ 토큰 리스트를 Vocabulary에 추가합니다.
      매개변수:
        tokens(list): Vocabulary에 추가할 문자열 토큰 리스트
      반환값:
        indices(list): 토큰 리스트에 상응되는 인덱스 리스트
      """
      return [self.add_token(token) for token in tokens]
    

    def lookup_token(self, token):
      """ 토큰에 대응하는 인덱스를 추출합니다.
      토큰이 없으면 UNK 인덱스를 반환합니다.
      매개변수:
        token(str): 찾을 토큰
      반환값:
        index(int): 토큰에 해당하는 인덱스
      노트:
        UNK 토큰을 사용하려면 (Vocabulary에 추가하기 위해) `unk_index`가 0보다 커야 합니다.
      """
      if self.unk_index >= 0:
        return self._token_to_idx.get(token, self.unk_index)
      else:
        return self._token_to_idx[token]
    

    def lookup_index(self, index):
      """ 인덱스에 해당하는 토큰을 반환합니다.
      매개변수:
        index(int): 찾을 인덱스
      반환값:
        token(str): 인덱스에 해당하는 토큰
      에러:
        KeyError: 인덱스가 Vocabulary에 없을 때 발생합니다.
      """
      if index not in self._idx_to_token:
        raise KeyError("Vocabulary에 인덱스(%d)가 없습니다." % index)
      return self._idx_to_token[index]


    def __str__(self):
      return "<Vocabulary(size=%d)>" % len(self)
    

    def __len__(self):
      return len(self._token_to_idx)
```

#### ReviewVectorizer
텍스트 데이터를 벡터의 미니배치로 만드는 두번째 단계는 입력 데이터 포인트의 토큰을 순회하면서 각 토큰을 정수로 바꾸는 것이다. `ReviewVectorizer` 클래스는 토큰과 인덱스가 매핑되어 있는 `Vocabulary` 객체의 lookup 테이블을 참고하여 입력받은 리뷰 텍스트를 one-hot 벡터로 변환한다. 
```python
class ReviewVectorizer(object):
  """ 텍스트를 수치 벡터로 변환하는 클래스 """
  def __init__(self, review_vocab, rating_vocab):
    """
    매개변수:
      review_vocab(Vocabulary): 단어를 정수에 매핑하는 Vocabulary
      rating_vocab(Vocabulary): 클래스 레이블을 정수에 매핑하는 Vocabulary
    """
    self.review_vocab = review_vocab
    self.rating_vocab = rating_vocab
  
  
  def vectorize(self, review):
    """ 리뷰에 대한 원-핫 벡터를 만듭니다.
    매개변수:
      review(str): 리뷰
    반환값:
      one_hot(np.ndarray): 원-핫 벡터
    """
    one_hot = np.zeros(len(self.review_vocab), dtype=np.float32)

    for token in review.split(" "):
      if token not in string.punctuation:
        one_hot[self.review_vocab.lookup_token(token)] = 1
    
    return one_hot
  

  @classmethod
  def from_dataframe(cls, review_df, cutoff=25):
    """ 데이터셋 데이터프레임에서 Vectorizer 객체를 만듭니다.
    매개변수:
      tokens(list): Vocabulary에 추가할 문자열 토큰 리스트
    반환값:
      indices(list): 토큰 리스트에 상응되는 인덱스 리스트
    """
    review_vocab = Vocabulary(add_unk=True)
    rating_vocab = Vocabulary(add_unk=False)

    # 점수 추가
    for rating in sorted(set(review_df.rating)):
      rating_vocab.add_token(rating)
    
    # count > cutoff인 단어 추가
    word_counts = Counter()
    for review in review_df.review:
      for word in review.split(" "):
        if word not in string.punctuation:
          word_counts[word] += 1
    
    for word, count in word_counts.items():
      if count > cutoff:
        review_vocab.add_token(word)
    
    return cls(review_vocab, rating_vocab)
  

  @classmethod
  def from_serializable(cls, contents):
    """ 직렬화된 딕셔너리에서 ReviewVectorizer 객체를 만듭니다.
    매개변수:
      contents(dict): 직렬화된 딕셔너리
    반환값:
      ReviewVectorizer 클래스 객체
    """
    review_vocab = Vocabulary.from_serializable(contents['review_vocab'])
    rating_vocab = Vocabulary.from_serializable(contents['rating_vocab'])

    return cls(review_vocab=review_vocab, rating_vocab=rating_vocab)
  
  
  def to_serializable(self):
    """ 캐싱을 위해 직렬화된 딕셔너리를 만듭니다.
    반환값:
      contents(dict): 직렬화된 딕셔너리
    """
    return {'review_vocab': self.review_vocab.to_serializable(),
            'rating_vocab': self.rating_vocab.to_serializable()}
```

#### DataLoader
파이프라인의 마지막 단계는 벡터로 변환한 데이터 포인트를 모으는 것이다. 파이토치의 내장 클래스인 `DataLoader`는 신경망 훈련에 꼭 필요한 미니배치로 모으는 작업을 편하게 해준다.
```python
def generate_batches(dataset, batch_size, shuffle=True, drop_last=True, device="cpu"):
  """
  파이토치 DataLoader를 감싸고 있는 제너레이터 함수입니다.
  각 텐서를 지정된 장치로 이동합니다.
  """
  dataloader = DataLoader(dataset=dataset, batch_size=batch_size,
                          shuffle=shuffle, drop_last=drop_last)
  
  for data_dict in dataloader:
    out_data_dict = {}
    for name, tensor in data_dict.items():
      out_data_dict[name] = data_dict[name].to(device)
    yield out_data_dict
```
