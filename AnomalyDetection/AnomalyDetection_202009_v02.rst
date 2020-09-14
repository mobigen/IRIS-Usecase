이상 탐지 ( Anomaly Detection ) - 2
===========================================================


모비젠의 Anomaly Detection 엔진
---------------------------------------------------------

특징
''''''''''''''''''''''

* 대량의 system 과 system 의 수많은 개별 feature 들의 임계치 자동 설정
* 주기가 있는 시계열 데이터의 이상 탐지
* 이벤트 로그 데이터(예: 장비의 알람 로그) , 센서 측정 데이터 등 다양한 데이터의 특성에 따른 이상 탐지
* System Heath Monitoring
    * system 의 개별 feature 의 이상 탐지 + system 전체의 status 변화를 정량적 수치화 
* Change-point Detection 
    * system 의 상태 변화 시점을 자동 탐지
* 이상 탐지 결과를 이용하여 Route Cause Analysis 에 적용


| 여기서는 모비젠의 "이상탐지 엔진" 에 적용된 기법들을 중심으로 설명하겠습니다.
| 모비젠의 "이상탐지 엔진(이하 ADE)" 에는 

.. code::

    기계학습 기반 알고리즘 + 회귀 분석 + 시계열 분석 기반 이상탐지

| 이 같이 적용되어 있습니다.



임계치 자동 설정
''''''''''''''''''''''

| 감시해야 할 센서, system의 수가 많거나





시계열 데이터의 이상 탐지
''''''''''''''''''''''''''''''''''''''''''

| 전력 사용량이나 이동 통신의 트래픽 사용량 처럼 주기성을 가진 시계열 데이터인 경우의 이상 탐지




| IRIS 의 ``이상탐지`` 메뉴에는 엔진의 알고리즘 일부가 적용되어 있습니다.





회귀 분석
''''''''''''''''''''''''''''''''''''''''''''



시계열 분석
''''''''''''''''''''''''''''''''''''''''''''


Anomaly Detection Techniques 개요
--------------------------------------



IRIS 의 "이상탐지"  기능
--------------------------------------

SPC 룰을 활용한 이상 탐지
''''''''''''''''''''''''''''''''


IQR 을 활용한 이상 탐지
''''''''''''''''''''''''''''''''


시각화 툴을 활용한 이상 탐지
''''''''''''''''''''''''''''''''




모비젠의 "이상탐지 엔진" 개요
--------------------------------------













**참고 문헌**

이상탐지, 시계열 분석 https://h3imdallr.github.io/2017-06-20/anomaly_detection/

anomaly detection 의 최신 트랜드 https://github.com/hoya012/awesome-anomaly-detection

한국보건사회연구원 정책보고서 https://www.kihasa.re.kr/web/publication/research/view.do?menuId=45&tid=71&bid=12&division=001&ano=2401

https://medium.com/@john_analyst/isolation-forest%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%9D%B4%EC%83%81%ED%83%90%EC%A7%80-%EB%AA%A8%EB%8D%B8-9b10b43eb4ac


참고 : 데이터 과학을 위한 R 알고리즘 https://statkclee.github.io/r-algorithm/r-mle-normal.html


