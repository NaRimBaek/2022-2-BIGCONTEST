# 2022_2_BIGCONTEST
앱 사용성데이터를 통한 대출신청 예측분석

# 목차
- 문제 정의
- 활용 데이터
- 데이터 전처리 및 EDA
- 분석과정 및 결과
- 핀다 앱 고객분석
-----------------------------------------------------------
- 분석 특징
- 내역할
- 어려웠던점, 해결방법
- 배운점/ 성과

# 문제 정의
  금융기관과 고객 사이의 정보의 비대칭성
  
  고객은 다양한 조건의 상품을 알기 어렵고, 금융기관 또한 고객이 서류를 제출하기 전까지 고객을 파악하기 어렵다. 
  
  -> 앱 사용성 데이터를 통한 대출신청 예측을 통해 고객의 선호도를 파악하여 고객에게 수요 맞춤형 상품을 추천할 수 있고, 시장에서 원하는 대출 상품을 개발할 수 있다. 
  
## 분석 목표

1. 대출상품 신청여부 예측
   핀다 앱 사용자 데이터 기반으로 어떤 고객이, 어떤 대출 상품을 신청할지 예측
   
2. 고객의 특성 분석
   대출신청, 미신청 고객을 분류하여 고객의 특성을 분석
   -핀다 앱에는 어떤 고객이 있는지, 대출신청한 고객과 안 한 고객의 차이는?, 대출신청한 고객들은 어떻게 나누어지는가?

# 활용 데이터

  * 유저 로그 데이터
    핀다 앱 유저가 앱을 사용한 활동 기록
    
  * 유저 스펙 데이터
    대출 신청서를 작성한 유저의 정보
    
  * 대출상품 결과 데이터
    신청서를 바탕으로 추천된 금융상품 목록과 해당 유저의 대출 신청 여부(target)
    
  * 외부 데이터
  
    -네이버 검색 데이터 : 성별, 연령별, 디바이스별 데이터
                      핀다, 대출, 신용대출, 금리비교 등 26개 단어 키워드
                      
    -COFIX : 코픽스 금리 데이터
    
    -CPI : 각 가정이 생활을 위해 구입하는 상품과 서비스의 가격변동을 나타내는 지수 데이터
    
    -공휴일 데이터 : 공휴일 데이터


# 데이터 전처리 및 EDA

## 이상치 처리
- 개인회생자 변수 : 개인회생자 여부를 묻는 항목이 핀다 어플에서 4월 18일 이후부터 생성되었기 때문에, 날짜를 고려해서 이상치 처리
- birth year & company enter month : 나이보다, 근속연수가 많다고 나타나는 경우, 이상치로 처리
- 승인한도, 승인금리가 결측인 경우 금융사에서 값을 보내지 않은 것으로 생각하고 분석에서 제외

## 결측치 처리
- 결측치 처리는 유저 스펙 데이터 내에서 처리가 가능한 경우와, 불가능한 경우로 나누어진다.

  즉, 동일 유저의 경우, 같은 유저의 이전 또는 이후에 해당 값이 있을 경우 그 값으로 대체 (test 데이터에서는 해당 일 기준 이전의 값으로만 대체)

- 유저 스펙 데이터 내에서 처리가 가능한 경우 먼저 처리한 다음, 이외 결측치는 논리적으로 추가 처리를 하였다. 

  ex) 신용 점수 : 유저별 평균으로 먼저 처리한 후, 남은 결측치는 XGBOOST modeling을 통해 처리

  
## log data의 "session" 나누기
- 유저가 30분 이상 어떤 활동도 하지 않는 경우, 세션은 종료되었다고 정의
- 자정이 되고 날짜가 바뀌는 순간 세션이 종료되었다고 정의
- 같은 유저가 같은 날, 같은 시각에 같은 행동이 연속적으로 여러 번 기록되어 있는 경우, 중복으로 기록된 것이라고 판단하여 제거 

  -> 이러한 session 이라는 기준을 활용하여 여러 파생변수를 생성

## 파생변수
  - loan_income_ratio : 소득에 비해 남은 대출금액을 나타내는 변수(기대출금액 / 연소득) 
   
  - product_popularity : 해당 상품의 선호도를 나타내는 변수. 
      (이전까지 해당 상품이 실제로 신청된 횟수 / 이전까지 해당 상품이 유저들에게 추천된 횟수)
      
  - Bank_popularity : 해당 금융사의 선호도를 나타내는 변수
      (이전까지 해당 금융사의 상품이 실제로 신청된 횟수 / 이전까지 해당 금융사의 상품이 유저들에게 추천된 횟수)
      
  - Loan_rate_rank : 동일 신청서 내에서 금리가 낮은 순서대로 정렬했을 때의 순위
  
  - Loan_limit_rank : 동일 신청서 내에서 대출 한도가 높은 순서대로 정렬했을 때의 순위

## EDA - 시계열성 특징이 존재한다. 

- 일별 대출 시청 & 승인 건수 시도표를 보면 일주일을 주기로 움직이는 시계열성을 보인다. 

- 휴일 다음날, 고점을 찍는 것을 확인할 수 있다. 

- 일별 대출 신청 건수/대출 승인 전체 건수 비율은 3월에서 5월로 갈수록 증가하는 추세를 보인다

      -> 요일을 의미하는 insert_time_weekday 변수 생성
      
      -> 공휴일 여부를 의미하는 is_holiday 변수 생성
      
      -> 시간이 갈수록 증가하는 추세를 반영하는 insert_time_month(1,2,3)변수를 추가
      


# 분석과정 및 결과

## model flow
<img width="676" alt="image" src="https://user-images.githubusercontent.com/78775413/219931772-49ac72e0-02f2-4b09-93a9-b52d5a67a471.png">

- random undersampling을 통해서 데이터 불균형을 어느정도 해소한 후, random search로 model의 parameter tuning을 진행
- Xgboost, Catboost, LGBM, Random forest, Logistic Regression, TabNET 모델들을 통한 분석 결과, LGBM 모델이 가장 결과가 우수했다. 


## 분석 결과
LGBM

<img width="254" alt="image" src="https://user-images.githubusercontent.com/78775413/219931828-00329a25-de4d-4b36-b0ac-3f91d58e79bd.png">

feature importance

<img width="729" alt="image" src="https://user-images.githubusercontent.com/78775413/219931898-b576df05-5d21-4460-aea6-83a015513a2b.png">


# 핀다 앱 고객분석

## 대출신청 유무, 대출신청서 작성 유무에 따른 고객 분석
- 대출신청서를 작성하지 않은 유저는, 대출신청서 첫 단계 부터도 실행하지 않는다. 

    -> 이 고객들을 유입하기 위해서는 처음 마주하는 화면에서부터 궁금해하는 것을 빠르게 제시하는 것이 필요하다
    
    -> 개인정보를 입력하지 않아도 상품 리스트를 간략하게 조회할 수 있는 페이지 추가할 것을 제안
    
- 신청서를 쓰고도 대출을 신청하지 않은 유저는 비교적 신용점수가 높고(비교적 간절하지 않은 상황이다), 희망금액과 승인한도의 차이가 크다. 오히려 승인 금리에서는 차이가 보이지 않았다. 



## K-means를 통한 핀다 앱 유저 세그멘테이션

<img width="852" alt="image" src="https://user-images.githubusercontent.com/78775413/219933197-b682a72e-a1fb-49b9-8e86-cb65b78f293d.png">

# 분석 특징

- 앱 내에서 어떤 고객이, 어떤 상품을 대출할지 예측하는 문제이기 때문에 실시간으로 앱에 가입하여 대출신청서를 작성하는 고객을 대상으로 이를 활용하기 위해서는 어느정도 모델이 가벼워서 빠르게 결과가 나와야 한다. 이 점을 고려하면 학습에 있어서 오래걸리는 딥러닝 모델보다, 비교적 빠른시간내에 학습을 마치고 결과를 업데이트 할 수 있는 LGBM 모델이 장점을 가진다고 생각한다. 

- EDA결과 시간이 지날수록 대출신청건수, 대출승인건수 대비 대출신청건수가 증가한다는 점을 확인하였고 이에 같은 유저라도 시간에 따라서 대출 신청유무가 차이가 난다는 점을 확인하였다. 즉 같은 유저라도 3월보다 5월에 대출신청서를 작성한 고객이 대출신청을 할 확률이 높아지고, 과거 데이터인 train(3-5)data를 가지고 모델을 학습시킨 후, test data인 6월 데이터에서 예측을 할 때 예상보다 대출신청을 하는 고객의 수도 증가할 것이므로 이를 분석에 반영하였다.


# 내 역할
전반적인 데이터 eda와 대출상품 신청여부 예측을 위한 모델 구현을 중점적으로 다루었다.
- 데이터 EDA를 통해서 대출 상품 신청건수의 시계열적 특성을 파악하고, 이를 분석에 구현하기 위해 고민하였다.(외부데이터 추가, 파생변수 생성 등)
- 대출 상품 예측 모델의 성능을 높이기 위해, random undersampling을 진행한 후, machine learning, deep learning 등 여러 모델들을 구현하였고, 최종적으로 LGBM MODEL을 선택하고 분석을 진행하였다.


# 어려웠던점, 아쉬운점

우리팀의 강점이라고 생각한 "시간의 특성을 반영한 분석"이 심사위원 평가에서 지적을 받았다. 
어떤 고객이, 어떤 상품을 대출할지 예측하는 문제인데, 왜 시계열 분석을 진행했고 시간적 특성을 모델에 왜 반영했는지에 대해 질문을 받았다. 
우리팀은 EDA결과 시간이 지날수록 대출신청건수, 대출승인건수 대비 대출신청건수가 증가한다는 점을 확인하였고 이에 같은 유저라도 시간에 따라서 대출 신청유무가 차이가 난다고 생각했다. 
즉 같은 유저라도 3월보다 5월에 대출신청서를 작성한 고객이 대출신청을 할 확률이 높아지고, 과거 데이터인 train(3-5)data를 가지고 모델을 학습시킨 후, test data인 6월 데이터에서 예측을 할 때 예상보다 대출신청을 하는 고객의 수도 증가할 것이라고 예측했다. 하지만 이 부분에 대해서 심사위원들에게 논리적으로 설명하지 못한 것 같아서 아쉬움이 남았다. 

train data에서 고객들 중에서 약 5%의 확률로 대출신청을 한 경우가 존재했는데(약 19:1의 비율), 대상 수상팀의 경우 다양한 모델들을 통해 분석한 후 그 중 예측 결과 대출신청한 사람들의 비율이 5%에 가장 가까운 모델을 최종 모델로 선택하였다. EDA결과 우리팀은 과거 데이터에서 대출신청 건수가 증가하는 추세를 확인할 수 있기 때문에, 5%보다 증가할 것이라고 생각했으며, 이러한 방식으로 최종 모델을 선택한 점에 대해 논리적으로 맞지 않다고 생각하여 아쉬웠었다. 


주어진 데이터가 방대하고, 매우 결측치가 많았기 때문에 data cleansing에 있어서 어려움이 존재했다. 
특히 log data의 경우 중복, 결측이 모두 많았기 때문에 이를 처리하는데 있어서 많은 시간과 노력을 쓰게 되었다. 예측 모델 생성에 있어서 앱 로그 데이터를 활용하기 위해 ”session”이라는 기준을 도입하여, 같은 유저가 같은 날, 시간대에 같은 행동이 여러 번 반복되는 경우 중복으로 판단하여 삭제하였고, 30분 이상 어떤 활동도 하지 않는 경우 세션이 종료되었다고 정의하였다. 이렇게 앱 로그 데이터를 세션을 기준으로 여러 변수들을 생성하여, 이를 분석에 반영할 수 있었다.



# 배운점, 성과

- 2022 빅 콘테스트 퓨처스리그 부분 본선진출

- Imbalanced data, 앱 log data 처리를 위한 다양한 방법들을 배울 수 있었다.

- 모델의 성능, 수치도 중요하지만, 분석 주제, 프로젝트 목적에 대해 다시한번 생각해보게 되었다. 

프로젝트 활동시에는 모델의 성능이 좋지 않아, 이를 높이기 위해 EDA를 통해 다양한 변수들을 생성하고 추세를 반영할 수 있는 외부데이터를 분석에 활용했었다.
좀 더 생각을 해보니 해당 프로젝트의 목적은 "어떤 고객이" "어떤 상품"을 대출하는가를 알고 싶겠다라는 생각이 들었다. 
모델의 성능이 비교적 조금 낮아도, 고객의 특징들을 주요 변수로 모델을 구현하는게 좀 더 좋은 점수를 받았을 것이라는 생각이 들었다. 
앞으로 예측분석에서는 성능향상도 중요하지만, 프로젝트의 목적에 집중하여 분석해야겠다는 점을 배울 수 있었다. 






