# 모델 학습 곡선 해석 가이드

지난 step 3에서 확인한 그래프들은 **모델 학습 과정**에서, **검증 데이터(eval set)에 대한 성능**을 나타내는 지표들입니다.  
TensorBoard에서 **`eval/f1`**, **`eval/accuracy`** 등을 시각화한 모습이며,  
이를 통해 **“현재 모델이 얼마나 잘 분류하고 있는지”**를 직관적으로 알 수 있습니다.

---

## 1. 기본 개념

### 1.1 정확도(Accuracy)

- **정의**  
  - 전체 예측 중, **정답을 맞힌 비율**을 의미
  - 수식으로는  
    \[
    \text{Accuracy} = \frac{\text{Number of correct predictions}}{\text{Total number of predictions}}
    \]
- **해석**  
  - 예: Accuracy = 0.47 (47%)라면, 모델이 100개의 쿼리 중 47개를 올바르게 분류한 것  

### 1.2 F1 점수(F1 Score)

- **정의**  
  - **정밀도(Precision)**와 **재현율(Recall)**의 조화평균
  - 분류 태스크에서 **불균형 데이터**나 **클래스별 분포**를 균형 있게 평가하기 위해 자주 사용
  - 식으로는  
    \[
    \text{F1} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}
    \]
- **해석**  
  - 주로 다중 클래스(3개 이상)일 경우 “가중치 평균(Weighted)” 방식 등을 사용  
  - Accuracy가 같은 두 모델이라도, **F1이 더 높은 모델**이 일반적으로 **클래스 간 편차 없이** 고르게 잘 맞추는 편  

### 1.3 학습 곡선

- **step**(혹은 epoch)  
  - 학습이 진행됨에 따라, 한 번의 epoch(또는 일정 batch step)마다 계산된 지표
- **Relative**(소요 시간)  
  - 학습 진행 시간(분/초 등)을 보여줌
- 일반적으로 **train** 곡선이 적절히 낮아지고, **eval** 곡선도 개선되면 모델이 잘 학습되고 있다고 판단

---

## 2. 그래프 해석

### 2.1 `eval/accuracy` 그래프
- 처음(50 step 전후)에는 0.44(44%) 부근  
- 마지막(150 step) 쯤에는 0.47(47%)로 상승  
- **해석**  
  - 학습이 진행되면서, 검증 데이터에 대한 정확도가 약간씩 오르는 추세  
  - 무작위 추측(3클래스 → ~33%)보다는 꽤 높은 편이나, 여전히 47% 정도로 실무에 쓰기에는 부족할 수 있음  
  - 더 많은 데이터나 하이퍼파라미터 튜닝 등을 통해 **정확도 개선**을 기대할 수 있음

### 2.2 `eval/f1` 그래프
- 초반 0.25~0.28 부근에서 시작  
- 후반(스텝 150)에는 0.3892 (약 0.39)  
- **해석**  
  - 다중 클래스 분류에서 **가중치 평균 F1**이 점차 상승한다는 것은, 모델이 고르게 성능을 높여가고 있음을 의미  
  - 여전히 0.39 수준이면, **개선 여지**가 많다고 볼 수 있음  
  - 만약 특정 클래스가 많이 틀리고 있다면, F1 향상이 느릴 수 있음 → **클래스별 정밀도/재현율** 분석 필요

---

## 3. 종합 요약 & 개선 제안

1. **학습이 정상적으로 진행**  
   - `train/loss`와 `eval/loss`가 모두 하락, `eval/f1`과 `eval/accuracy`가 서서히 오르는 패턴을 확인할 수 있음  
   - 이는 모델이 과적합 없이 점차 학습 중임을 시사

2. **아직 절대 성능은 낮을 수 있음**  
   - 정확도가 ~47%, F1이 ~0.39 수준이라면, **실제 서비스**에 적용하기엔 부족  
   - 다만, 학습 데이터셋 규모, 태스크 난이도, 라벨 품질 등에 따라 달라질 수 있음

3. **개선 방안**  
   - **에폭 수 증가**: 예) 지금 3에폭이라면 5~10에폭으로 늘려보기  
   - **데이터 품질/양 개선**: 불균형 데이터, 노이즈 라벨 등을 점검  
   - **모델 변경**: 더 큰 사이즈의 BERT 계열, ModernBERT(`answerdotai/ModernBERT-base`), RoBERTa 등  
   - **하이퍼파라미터 튜닝**: 러닝레이트, 배치 사이즈, 옵티마이저 스케줄러 등  
   - **클래스별 상세 분석**: Confusion Matrix, Precision/Recall by class 등을 보면 특정 클래스에서 극단적으로 오분류가 일어나는지 파악 가능

---

## 4. 추가 팁

- **TensorBoard**에서  
  - “Scalars” 탭을 보면, 여러 지표(`loss`, `accuracy`, `f1`, `grad_norm`, `learning_rate`)를 확인  
  - “Graphs” 탭에서는 모델 구조를 살펴볼 수 있음(Transformers는 간혹 제한적일 수 있음)  
- **클래스별 지표**  
  - `Trainer`의 기본 compute_metrics 함수 대신, `precision_recall_fscore_support`(sklearn) 등을 사용해 클래스별 F1을 구해 보기를 권장  
- **지표 변동 폭**  
  - 학습 초반에는 지표가 급격히 변동할 수 있으나, 에폭이 지나면서 점차 완만해짐  
  - smooth 기능으로 평균선을 보면 추세 확인에 좋음

---

## 결론

- **Accuracy**와 **F1**가 모두 상승 추세이므로 모델이 학습 중임을 알 수 있고,  
- 최종 결과로 47% 정확도, 0.39 F1은 **베이스라인** 수준으로 볼 수 있음.  
- **더 높은 성능**을 원한다면, **더 많은 에폭**, **데이터 증량**, **하이퍼파라미터 최적화**, **다른 모델** 등을 시도하여 **지속 개선**이 필요합니다.