# 2020년 가을학기 금융데이터분석 

## 주제: 실물경제 데이터와 금융데이터의 기계학습을 통한 주가예측
 
### 프로젝트 세분화
 ![image](https://user-images.githubusercontent.com/28617435/123709291-cc150600-d8a7-11eb-988f-64a2b572c0c6.png)
 
 (1) **기계학습 모델링**
  - agent 모델을 기반으로 모델링을 실시하였다. 투기거래자는 Fundamental에 기초하지 않고 오직 투기심리에 의하여 투자를 하는 반면 Fundamental 거래자는 국민소득에 상응하게 투자를 하는 것을 바탕으로 모델을 구성했다. 투기거래자의 심리를 반영하는 데이터로 채권의 yield spread를 이용하였으며 Fundamental 거래자의 경우 GDP에만 반응하여 투자한다고 가정하여 GDP의 각 구성요소들을 활용한다. GDP 구성요소 중 순 수출의 비중은 다른 요소에 비해 매우 작은 비중을 차지하기 때문에 본 모델에서는 생략한다.
  - 활용데이터: 
   > 1) Personal Consumption 항목의 경우 Durable Goods, Non-Durable Goods, Services (
   > 2) Investment 항목의 경우 Business investment, Fixed Investment 
   > 3) 정부 지출 항목의 경우 Government Total Spending 
   > 4) GDP(Annual %), 채권 Yield Spread 
   > 5) 종속변수 라벨링: S&P500의 미래 3개월 수익률이 0이하면 Negative, 0에서 2.4 사이면 moderate, 2.4 이상은 positive로 구분하였다.
   > (출처: Fred)

  - 모델: DecisionTreeRegressor
   > 1) Xgboost, RandomForest 등을 Cross_Validation으로 여러 머신러닝 모델에 직접 대입해보며 가장 성능이 좋았던 DecisionTreeRegressor를 선정
   > 2) 이때 정확도는 75%, 가장 유의미한 변수로는 10년 만기 채권물이 선정되었다.


 (2) **최적화**
  - 평균-분산 모형을 기반으로 최적화를 하였다. 추가한 제약식은 아래와 같다.
  
    ![image](https://user-images.githubusercontent.com/28617435/123712165-bce48700-d8ac-11eb-9624-4a635f122cc8.png)

   > 1) 공매도 금지
   > 2) Turnover(이전 자산 구성율과 변화 정도에 대한 제약)
   > 3) Rank: 만약, (1)번의 머신러닝 모델 결과 수익률이 positive로 측정되었다면, 가장 높은 수익률의 투자 비중은 최하위 50% 자산보다 많은 비중을 가져가며, 가장 낮은 수익률의 투자비중은 최상위 50%보다 낮은 비중을 가져가도록 함
   > 4) 채권비중: 투자자 성향을 측정한 후, 공격형/중립형/안정형에 따라 채권 비중을 조절

 (3) **Backtest** 
  - 투자자 입력사항: 투자자는 본 연구의 프로그램에 투자 시작일, 투자 종료일, 투자 금액, 투자하고자 하는 자산군, 투자성향에 관한 설문 문항, Lookback Period를 입력
   > - 여기서 투자 자산은 ‘Fama-French 5 Industry‘, ‘Fama-French 10 Industry‘ 중 1개를 선택하여 입력하며, 두 데이터프레임 모두 기존 ‘Others’ 자산을 미 국고채 10년물 채권으로 대체함
   > - 투자성향에 관한 5가지 설문에 대해 4점 척도를 활용하여 투자자에게 맞는 투자성향을 산출하고자 하였다. (김지영, 2011). 5가지 설문에서 사용자가 입력한 값을 토대로 점수의 총 합이 각각 “10이하, 10초과 15미만, 15이상” 일 때 안정형, 중립형, 공격형을 나누었다. 
  - 초기값: 채권을 제외한 나머지 투자 자산에 대해 1/N로 초기 투자 실시
  - 사용자가 입력한 Rebalance 주기에 따라 자산비중 재조절 (최적화)이 일어난다. 하지만 만약 마지막 투자 종료기간과 Rebalance 주기가 일치한다면 이 때는 Rebalance를 실시하지 않았다. (어차피 회수 예정으로 불필요한 비용 방지)

 (4) **UI**
  -  Price Graph: 시간이 지남에 따라 포트폴리오의 가치 변화를 1/n 포트폴리오와 S&P500과 비교하여 보여줌
  -  Return Graph: 최적화된 포트폴리오의 변동성을 1/n 포트폴리오와 S&P500과 비교하여 보여줌
  -  Asset Allocation Pie Charts: 투자 비중의 변화를 Rebalnce 주기에 따라 보여줌
  -  DrawDown: 포트폴리오의 DrawDown과 Maxmum DrawDown을 보여줌
  -  Return Histogram: 수익률 분포를 시각화하여 Fat-tail 부분의 유무를 파악하고 수익률의 전체적인 분포를 파악하는 히스토그램을 보여줌
  -  Performance Evaluation: 최적 포트폴리오의 성과를 Sharpe ratio 등 척도를 이용한 표를 보여줌
  - 시연영상: https://www.youtube.com/watch?v=1YhgB6fpxGQ

 (5) **결론 및 제언**
  - 본 연구는 머신러닝을 활용한 경제 상황 학습 및 예측과 투자자의 투자 성향(안정형, 위험 중립형, 공격 투자형)에 따라 최적화된 포트폴리오와 Backtest 결과를 보여주는 프로그램을 설계하였다. 
  - 시각화 자료나 표를 참조하면 알 수 있듯 이는 본 프로그램이 기반으로 하는 포트폴리오 최적화 방법론인 ”Mean-Variance Model“이 주장하는 Return과 Volatility의 성질을 잘 드러냈다.

 (6) **한계**
  - 한계1. 고정적인 Rebalance 주기: 사용자는 Rebalance 주기를 정하면 중간에 수정할 수 없게 구현한 것이 아쉬운 포인트다.
  - 한계2. 투자 성향에 대한 세분화 필요성: 현재는 공격/중립/안정 3가지로만 투자자를 구분한다. 현실에서는 공격 투자형이나 안정형에 지극히 편향된 투자를 하는 경우가 드물며 위험 중립형처럼 양 측의 이점을 분배하여 투자함으로써 효율적으로 투자하고자 한다. 그러나 본 연구는 이러한 중립적인 투자 성향을 1가지만 고려하여 투자자 선택의 폭을 줄였다는 한계가 있다.

 (6) **참고문헌**
  - 김지영 (2011), 개인투자자의 투자성향에 따른 금융자산 포트폴리오, 성신여자대학교 	석사학위논문
  - 윤재호 (2020), 이자율 스프레드의 경기 예측력: 문헌 서베이 및 한국의 사례 분석, 	한국은행 경제연구원 보고서
  - Frank Westerhoff (2012), Interactions between the Real Economy and the stock Market: 	A Simple Agent-Based Approach, University of Bamberg
