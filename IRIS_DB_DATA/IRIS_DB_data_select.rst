================================================================================
IRIS DB 기반의 데이터 
================================================================================
    
| IRIS Studio 에서 주로 사용되는 **데이터 유형** 에는  "데이터 모델", "DSMS"  가 있습니다.

- 데이터 모델 
    - 데이터를 추상화한 데이터 묶음으로서 IRIS 내에서 별도의 테이블처럼 사용할 수 있습니다.  RDBMS, IRIS DB, OBJECTSTORAGE, HDFS 의 데이터를 데이터 모델로 생성할 수 있습니다.
    - `데이터 모델 manual <https://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery/datamodel.html#id1>`__  이동

- DSMS
    - Data Source Manager Service, 다양한 데이터에 대한 연결정보를 제공하여 해당 데이터 소스에 맞는 검색어를 사용합니다. 
    - `DSMS manual <https://docs.iris.tools/manual/IRIS-Manual/IRIS-Studio/studio/index.html?highlight=dsms#id10>`__ 이동


| 여기서는 IRIS DB 테이블 중 **로컬 테이블** 데이터를 대상으로 만든 "데이터 모델"  과 "DSMS" 에 대해 설명합니다.
| IRIS DB 의 로컬 테이블 데이터를 IRIS 검색어로 효율적으로 조회하기 위해서는 IRIS DB 의 특징에 맞춘 별도의 방법이 필요합니다.
|
| IRIS DB 는 대용량의 시계열 데이터를 실시간으로 고속 저장, 검색을 할 수 있는 대용량 분산 DB 입니다.
| 14자리 timestamp 타입의 컬럼을 기준으로 N분, N시간, N일, N달, N분기, N년(year) 등 으로 PARTITION 을 생성하여 대량의 데이터를 고속으로 분산 처리하여 입력합니다.
| 그리고 데이터를 조회할 때에는 LOCATION HINT 를 통해 조회 대상 PARTITION 을 지정하여 빠른 검색을 할 수 있습니다.



------------------------------------------------------------
데이터 모델 
------------------------------------------------------------

''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Discovery 에서 데이터 모델 생성하기
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ``IRIS Discovery >> 데이터 모델`` 의 목록에서 "데이터 유형" 필드의 값이 **IRIS**  인 경우의 데이터 모델에 해당합니다.
| IRIS DB 와의 연결정보를 통해 IRIS DB 테이블을 지정한 후 검색어 결과로 나온 데이터 셋을 데이터 모델로 생성합니다.

.. image:: ../images/IRIS_DB_DATA/datamodel_01.png
    :alt:  IRIS 데이터모델 01


| 데이터 모델 생성 시에 **시간** 컬럼에 해당 IRIS DB 테이블의 Partition 생성 기준이 되는 시간 컬럼을 지정합니다. 그리고 YYYYMMDDHHmmss 로 포맷을 지정합니다.



''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Studio 에서 데이터 모델 유형으로 데이터 객체 생성하기
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


.............................................
데이터 객체의 시간 설정
.............................................

| ``시간 선택`` 에서 조회 시간을 "미리 설정" 또는 "날짜 및 시간범위" 를 지정하거나 ``객체 연결`` 을 사용합니다. 

- 시간 선택 : 미리 설정(전체 선택 제외) 
    - ``미리 설정`` 목록에서 지정을 합니다. 이 때 데이터의 양, PARTITION range( 1시간/1일/1주/1달/...) 에 따라 적절한 시간 목록을 선택해야 합니다. 
        - 미리 설정 시간 예시 :  최근 60분, 최근 4시간, 최근 24시간, 최근 7일
    - 선택한 시간에 해당하는 PARTITION 을 조회 대상으로 합니다. 해당 PARTITION 내에서 시간 범위를 지정할 경우에는 검색어 쿼리에 **where** 절에 PARTITION range 를 정하는 시간 컬럼으로 상세 범위를 설정합니다.
   
.. code-block::

    # 예) "미리 설정 - 최근 24시간" 으로 선택, 실제 가져올 데이터의 시간은 24시간 보다 작은 범위의 데이터 일 때
    # 시간 선택           : 최근 24시간 ( 2021/10/05 02:05:23 ~ 2021/10/06 02:05:22 )
    # 대상 데이터 조회 시간 : 2021/10/05 03:40:00 ~ 2021/10/05 23:10:00

    * |  where PRERGSTDT >= '20211005034000'  and PRERGSTDT <= '20211005231000' 


- 시간 선택 : 미리 설정 - 전체 선택
    - "전체 선텍"  은 객체 연결과 같이 **이벤트 변수** 를 받아서 처리하는 경우에만 선택하고, 이벤트 변수로 받은 시간을 검색어 구문 제일 앞에 넣어야 합니다.
    - 아래 처럼 작성한 검색어 구문은 start_date, end_date 에 해당하는 PARTITION 만을 대상으로 검색합니다.

.. image:: ../images/IRIS_DB_DATA/datamodel_03.png
    :alt:  IRIS 데이터모델 03


.. code-block::

    * start_date= 20210928000000   end_date= 20211005102712 
    | where CNTRCTCNCLSMTHDNM != '수의계약'


- 객체 연결
    - 시간 선택 목록에서 알맞은 목록이 없거나, 트리거 이벤트로 시간 변수를 받아서 처리할 경우에는 ``객체 연결`` 을 사용합니다. 
    - 객체 연결을 사용하기 위해서는 ``기간설정`` 객체 등을 통해 **이벤트 변수** 로 시작 시간, 끝 시간을 입력 받아야 합니다.
    - 데이터 객체의 ``실행 방법 설정`` 은 트리거를 설정합니다. ``기간 설정`` 객체에서 이벤트 변수를 받는 경우에는 ``버튼`` 객체를 클릭하는 이벤트로 트리거를 설정합니다.
    - ``기간설정`` 객체 이용
        - ``기간설정`` 객체의 변수 ``${date_time_range_picker_1.startDate}``, ``${date_time_range_picker_1.endDate}`` 를  ``객체 연결`` 에서 시작 시간, 끝 시간으로 설정합니다.
        - 객체 연결에서 사용할 수 있도록 ``기간설정`` 변수의 시작 시간, 끝 시간 변수값 포맷은 **YYYYMMDDHHmmss**  로 설정해야 합니다.
        
.. image:: ../images/IRIS_DB_DATA/datamodel_02.png
    :alt:  IRIS 데이터모델 02



------------------------------------------------------------
DSMS  
------------------------------------------------------------

''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Studio 에서 DSMS 유형으로 데이터 객체 생성하기
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


.............................................
데이터 객체의 검색어 구문
.............................................

| IRIS DB 테이블 중 로컬 테이블 조회시에는 검색어 구문에 조회 시간을 LOCATION HINT 에 넣어야만, 조회 시간에 해당하는 PARTITION 만을 대상으로 빠른 검색이 가능합니다. `IRIS 로컬 테이블에서 LOCATION HINT 사용하기 <https://docs.iris.tools/manual/IRIS-Manual/IRIS-Database/user_guide/doc/01.query.html#location-hint>`__
|

.. image:: ../images/IRIS_DB_DATA/datamodel_04.png
    :alt:  IRIS 데이터모델 04


- LOCATION HINT 사용 검색어 예시 ( 1시간 단위 PARTITION 일 때 )
    - 데이터 조회 시간 : 2021/10/05 03:40:00 ~ 2021/10/05 23:10:00
    - LOCATION HINT : PARTITION '20211005030000',  '20211005040000',,, '20211005230000' 를 대상으로 합니다.


.. code-block::

    # LOCATION HINT 로 PARTITION 범위를 지정한 후, 상세 조회 시간을 where 절에 추가합니다.

    /*+ LOCATION ( PARTITION >= '20211005030000'   AND PARTITION <= '20211005235959' ) */
    select * from EPS.EPS_DATA 
    where PRERGSTDT >= '20211005034000' 
      and PRERGSTDT <= '20211005231000' 
      and CNTRCTCNCLSMTHDNM != '수의계약'

   # partition key 가 설정된 경우
   /*+ LOCATION (KEY = '2' AND PARTITION >= '20150101000000' AND PARTITION < '20150102000000') */
   select * from EPS.EPS_DATA 
    where CNTRCTCNCLSMTHDNM != '수의계약'


- 참고 : IRIS LOCAL 테이블 생성 예시

::

    CREATE TABLE LOCAL_TEST_TABLE (
       k         TEXT,
       p         TEXT,
       a         TEXT
    )
    datascope       LOCAL
    ramexpire       30
    diskexpire      34200
    partitionkey    k
    partitiondate   p
    partitionrange  10
    ;

- IRIS 테이블 옵션


IRIS에서는 테이블을 생성시 테이블의 타입 및 보관 주기등을 설정해 주어야 합니다.

-  datascope : 데이터 저장 방식 설정 [ LOCAL OR GLOBAL ]
-  ramexpire : 램에 저장하는 시간 [ GLOBAL 테이블일 경우 0 ]
-  diskexpire : 디스크에 저장하는 시간 [ GLOBAL 테이블 경우 0 ]
-  partitionkey : KEY 로 사용할 값이 저장되는 컬럼 [ GLOBAL 테이블 경우 None ]
-  partitiondate : PARTITION으로 사용할 값이 저장되는 컬럼 [ GLOBAL 테이블 경우 None ]
-  partitionrange : 하나의 PARTITION 이 가지는 범위 [ GLOBAL 테이블 경우 0 ]
