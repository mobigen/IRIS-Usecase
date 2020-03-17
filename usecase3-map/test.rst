=======================================================================================================================
IRIS Studio - DEMO_서울시 안전센터_소방서 위치 및 화재 통계
=======================================================================================================================

IRIS Studio 를 이용하여
 `서울시 열린 데이터 광장 <https://data.seoul.go.kr/dataList/datasetList.do>`__ 에 있는 
서울시 안전센터 관할 위치 정보와 서울시 소방서 관할 위치 정보와 서울시 원인별 화재 통계 데이터를
지도와 챠트로 만들어 봅니다.


.. image:: ../images/demo/demo_fire_01.png
    :alt: 데이터 - 01 


.. contents::
    :backlinks: top


------------------------------
데이터 준비
------------------------------

''''''''''''''''''''''''''''''''
데이터 가져오기 
''''''''''''''''''''''''''''''''

- 출처 : `서울시 열린 데이터 광장 <https://data.seoul.go.kr/dataList/datasetList.do>`__ 
    - 보고서의 상단 오른쪽 라벨을 클릭하면 출처인 서울시 열린 데이터 광장으로 이동합니다.

- 출처
    - 서울시 안전센터, 소방서, 화재 원인 데이터 
        - 서울시 열린데이터 광장 `서울시 열린 데이터 광장 <https://data.seoul.go.kr/dataList/datasetList.do>`__ 
    - 서울시 구단위 행정 경계 데이터 ( SHP 파일 )
        - 국토교통부 행정구역도 `공공 데이터 포털 <https://www.data.go.kr/dataset/3046391/openapi.do>`__


'''''''''''''''''''''''''''''''''''
데이터 업로드
'''''''''''''''''''''''''''''''''''

- 로컬 PC 에 다운로드된 파일들을 IRIS 의 **HDFS브라우저** 를 이용하여 MinIO 에 업로드합니다.

`MinIO 에 데이터 업로드 <http://docs.iris.tools/manual/IRIS-Usecase/usecase4-batting_data/DEMO_batting.html#minio>`__


.. list-table::
    :header-rows: 1

    * - 모델 이름
      - 설 명  
    * - SEOUL_SAFETY_CENTER_COOR
      - 서울시 안전센터 위치
    * - SEOUL_MELT_FIRE_CAUSE
      - 서울시 소방서별 화재원인 분포
    * - SEOUL_GU_WGS84
      - 서울 행정구 경계 polygon WGS_84
    * - SEOUL_GU_FIRE_CAUSE
      - 서울시 구별 화재원인 통계
    * - SEOUL_GU_COORDINATES
      - 서울 구청 좌표 및 정보
    * - SEOUL_FIRE_STA_COOR
      - 서울시 소방서 좌표
    * - SEOUL_FIRE_CAUSE
      - 서울시 관할 소방서별 화재원인(2011 ~ 2018년)