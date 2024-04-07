## 치매 고위험군 분류를 위해 데이터 전처리 시도 (걸음걸이 데이터 사용)




연구 대상 및 자료 수집 절차 (ai hub데이터만) 

AI Hub 사이트를 통해 치매 고위험군의 데이터를 수집했습니다. 연구의 대상은 다음과 같습니다.
연령: 모두 55세 이상의 대상자로, 50대 4명, 60대 63명, 70대 193명, 80대 이상 40명으로 구성되었습니다.
건강 상태: 정상인, 경도 인지 장애, 치매 전 단계의 약 300명을 모집했습니다.
데이터 밸런스: 정밀진단 정상인(CN) 150명, 치매 위험군(무증상(aAD), 전조증상(MCI), 치매(AD)) 150명으로 구성되었습니다.
성별: 남성 137명, 여성 163명으로 분배되었습니다. 




## 데이터 구조




모든 파일은 csv파일로 저장되어있으며 구조는 아래를 따릅니다.

![image](https://github.com/mok010/dementia_classification/assets/76732607/15159c74-4ac1-4d73-aba3-063ad1b9f0a9)


![image](https://github.com/mok010/dementia_classification/assets/76732607/008d5d76-1afe-4072-9239-029c9b6358b9)


## 헬스 커넥트 쓸만한 센서 관련 클래스 



ActiveCaloriesBurnedRecord
기초 대사율(BMR)을 제외하고 사용자가 소모한 추정 활성 에너지(킬로칼로리)를 캡처합니다. 각 기록은 시간 간격 동안 소모된 총 킬로칼로리를 나타내므로 시작 시간과 종료 시간을 모두 설정해야 합니다.
BodyTemperatureRecord
휴식 중(예: 잠에서 깬 직후) 사용자의 체온을 캡처합니다. 각 데이터 포인트는 단일 순간 체온 측정값을 나타냅니다.
DistanceRecord
마지막 판독 이후 사용자가 이동한 거리를 캡처합니다. 특정 구간의 총 거리는 해당 구간 동안의 모든 값을 합산하여 계산할 수 있습니다. 각 기록의 시작 시간은 해당 거리가 포함된 간격의 시작을 나타내야 합니다.

장기 운동 시나리오에서 분석을 선호하는 경우 여러 거리 기록을 작성하는 것이 좋습니다. 각 레코드의 시작 시간은 이전 레코드의 종료 시간과 같거나 커야 합니다.
ExerciseRoute
사용자가 수행하는 운동 세션과 관련된 경로를 캡처합니다.
순서대로일 필요는 없는 타임스탬프와 함께 일련의 위치 지점을 포함합니다.
위치 포인트에는 타임스탬프, 경도, 위도 및 선택적으로 고도, 수평 및 수직 정확도가 포함됩니다.
ExerciseSegment
운동 세션 내의 특정 운동을 나타냅니다.
각 세그먼트에는 운동 시작 및 종료 시간, 운동 유형 및 선택적 반복 횟수가 포함됩니다. => 러닝데이터
HeartRateRecord
사용자의 심박수를 측정하는 클래스
HeartRateVariabilityRmssdRecord
심박 변이도를 측정하는 클래스
RestingHeartRateRecord
사용자의 안정시 심박수를 캡처합니다. 각 기록은 단일 순간 측정을 나타냅니다.
SleepSessionRecord
사용자의 수면 시간과 단계를 캡처하는 클래스
-깨어있는지 여부
-깨어있으며 침대에 있는지 여부
-깨어있으며 침대 밖에 있는지 여부
-deep sleep단계 여부
-light sleep 단계 여부
-rem sleep 단계 여부
-수면 단계를 알수 없음

SpeedRecord
사용자의 속도를 캡처합니다. 달리거나 자전거를 타는 동안. 각 기록은 일련의 측정값을 나타냅니다.
-평균, 최대, 최소
StepCandenceRecord
사용자의 걸음 수를 캡처합니다. 각 기록은 일련의 측정값을 나타냅니다.
-평균, 최대, 최소
StepsRecord
사용자의 걸음 수를 측정하는 클래스
-총 걸음수
TotalCaloriesBurnedRecord
활성 및 기초 소모 에너지(BMR)를 포함하여 사용자가 소모한 총 에너지(킬로칼로리)입니다. 각 기록은 일정 기간 동안 소모된 총 킬로칼로리를 나타냅니다.


## 헬스 커넥트에서 데이터를 가져오는 방식


헬스 커넥트에서 가져 올수 있는 데이터는 4가지. 
심박수와 같은 통계적 데이터 (Statistical), 달에 뛴 횟수와 같은 세기 데이터 (Count), 걸음수와 같은 누적 데이터 (Comulative), 딥슬립의 시간과 같은 기간 데이터 (Duration)이다.

통계적 데이터 (Statistical)
세기 데이터 (Count)
누적 데이터 (Comulative)
기간 데이터 (Duration)

이는 각각의 목록을 가지고 불러와질 수 있다.

문서에서의 내용을 참고하자면 데이터를 가져오는 방식은

1. 데이터를 레코드를 구조화. => 즉 가져오고 싶은 데이터들을 모아 구조체를 만든다.
2. 그후 insertRecords를 사용하여 데이터 쓰기를 시도한다,
3. 데이터가 헬스 커넥트에 쌓인다.
4. readRecords를 사용하여 시작시간과 마침시간을 입력한 것을 기준으로 구조체의 리스트 형태로 데이터를 받아온다.

## 구조체의 리스트를 csv 파일로 변환하는 방법

구조체 리스트를 csv 파일로 변환하기 위해서는 먼저 구조체리스트를 데이터 프레임화 시켜주고 그 후에 이를 csv 파일로 변환할 필요성이 있다. 

## Work Flow Diagram 

![team  날개 스마트워치를 활용한 치매 고위험군 분류 모델](https://github.com/mok010/dementia_classification/assets/76732607/b71835d3-a520-409c-9528-b841b6201e20)

## 안드로이드 앱 샘플

https://ovenapp.io/view/b3zamLxubR4h8A70FHwrAs9RV0g08GbO/


