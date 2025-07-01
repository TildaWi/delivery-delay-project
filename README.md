# 이상 탐지 기반 '배송 지연 패턴 분석' 프로젝트 🚚

### 분석 카테고리: 데이터 활용 모델링
> **분석 기간** &nbsp;|&nbsp;  2025.06 - 2025.06 <br/>
> **분석 주체** &nbsp;|&nbsp;  개인 프로젝트 <br/>
> **분석 기법** &nbsp;|&nbsp;  이상 탐지, 시계열 분석, 패턴 분석 <br/>
> **분석 기술** &nbsp;|&nbsp;  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white"/> <img src="https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white"/> <img src="https://img.shields.io/badge/Numpy-013243?style=flat&logo=numpy&logoColor=white"/> <img src="https://img.shields.io/badge/Matplotlib-11557C?style=flat&logo=matplotlib&logoColor=white"/>

---

## 0. 들어가기 전에

### 📂 디렉토리 구조

```plaintext
📁 delivery_delay_project/
 ┣ 📁 data/              원천 데이터 (Delhivery 배송 이력, 지역, 시간 등)
 ┣ 📁 notebooks/         분석 코드 및 이상 탐지 모델링 (Colab)
 ┣ 📁 images/            시각화 결과 (이상 점수 분포, 조건별 비교 차트 등)
 ┣ 📁 reports/           보고서 및 발표자료 (PDF, PPT)
 ┣ 📄 README.md          프로젝트 설명 문서
 ┗ 📄 requirements.txt   사용한 Python 패키지 목록
```

---
## 1. 프로젝트 개요

### 📌 세 줄 요약
- 배송 지연은 단순한 시간 초과가 아닌 고객 신뢰도와 직결되는 핵심 문제로, 사전 예측 기반 대응이 필요합니다.  
- Delhivery 물류 데이터(144,867건)를 기반으로 이상 탐지 기법을 조합하여 고위험 배송을 분류하고, 대응 체계를 설계했습니다.  
- 이상 탐지 결과를 활용한 실시간 알림 시스템과 운영 대응 매뉴얼을 구축하여 배송 지연 최소화 전략을 제시했습니다.

---

## 2. 문제정의 및 접근방식

### 🔍 Situation
- 배송 운영 담당자가 실시간으로 지연 상황을 판단하기 어렵고, 외부 변수(날씨, 이벤트 등)로 인해 지표 기준이 일관되지 않음

### 💡 Task
- 이상 탐지 기법을 활용해 배송 지연을 사전에 감지하고, 이를 기반으로 실시간 대응 가능한 알림 시스템 및 매뉴얼 설계

### 🏃 Action
- Delhivery 물류 데이터 분석, 이상탐지 조건 정의, Z-score, Rule-based, Isolation Forest를 활용한 앙상블 탐지  
- 이상 점수 기반 분포 분석 → 고위험 출발지·시간대 특성 파악 → 단계별 대응 매뉴얼 및 운영 지표 설계

### 🚀 Result
- 전체 배송의 10.5%를 이상 배송으로 분류 (위험 단계 6.0%, 주의 단계 4.5%)  
- 이상 탐지 기준에 따른 경고 임계값 및 대응 매뉴얼 설계 완료 → 실시간 대응 시스템으로 확장 가능성 확보

---

## 3. 프로젝트 진행

### 3-1) 📊 EDA 요약

- 이상 그룹은 평균 배송 시간이 유의하게 길고, 심야 시간대(22시 전후) 또는 특정 출발지(Gurgaon, Bangalore 등)에 집중  
- 주요 변수: 배송 시간(`actual_time`, `osrm_time`), 거리, 출발지, 시간대, 운송 수단  

📊 **삽입 이미지**: `images/anomaly_distribution.png`  
📊 **삽입 이미지**: `images/delay_by_location_hour.png`

---

### 3-2) 🧪 가설 검정: 정상/이상 배송 그룹 비교

**H0 (귀무가설)**: 이상 배송과 정상 배송 간 평균 배송 시간에 유의미한 차이가 없다  
**H1 (대립가설)**: 두 그룹 간 평균 배송 시간에 유의미한 차이가 존재한다  

- **결과**: 평균 배송 시간, 출발지 분포, 시간대 등에서 유의미한 차이 존재 → H0 기각  

📊 **삽입 이미지**: `images/stat_test_summary.png`

---

### 3-3) 🤖 이상 탐지 기반 알고리즘 구성

#### 사용된 이상 탐지 알고리즘

- **Rule-based 조건식**  
  - `actual_time - osrm_time > 60분`  
  - `delay_ratio > 1.5`  
  - `Z-score > 3` 중 하나 이상 충족 시 '이상'으로 분류

- **Isolation Forest**  
  - 트리 기반 이상 탐지 알고리즘으로, 고차원 공간에서 고립되는 데이터를 이상치로 탐지

- **Z-score 기반 탐지**  
  - 통계적 이상치 판단 기준 (평균으로부터 3시그마 이상 벗어난 값 탐지)

- **Ensemble 방식 적용**  
  - 위 세 가지 알고리즘을 조합하여 이상 점수(`anomaly_score`)를 산출하고,
    점수 기반으로 `주의/위험 단계`를 분류


#### 이상 탐지 기반 분류 기준

##### 지연 기준 (Delay Rule)
- `actual_time - osrm_time > 60분`  
- `delay_ratio > 1.5`  
- `Z-score > 3`  
→ 위 조건 중 하나 이상 충족 시 '지연'으로 분류  

##### 🟡 **주의 단계 조건**
- 이상 탐지 조건 2개 충족  
- 이상 점수 5~10% 구간  
- 고위험 출발지 + 피크 시간대(예: 일요일, 오후 5시 등)

##### 🔴 **위험 단계 조건**
- 이상 탐지 조건 3개 이상 충족  
- 이상 점수 하위 5% (Quantile 기준)  
- 동일 배송에서 연속 지연 3회 이상 발생

📊 **삽입 이미지**: `images/anomaly_scoring_framework.png`

---

## 4. 프로젝트 회고

### ✏️ Learned Lessons
- 배송 지연은 단순한 결과가 아닌 **운영 구조와 데이터 기반 판단의 어려움이 복합된 문제**임을 실감  
- 이상 탐지 기법과 운영 지표를 결합해 **실시간 대응 가능한 구조 설계**가 가능하다는 것을 경험  
- 향후에는 **알림 시스템을 실제 시뮬레이션하고 운영 성과를 추적하는 고도화 방향**으로 확장이 필요함
