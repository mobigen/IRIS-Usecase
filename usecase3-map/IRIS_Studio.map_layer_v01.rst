.. sectnum::

================================================================================
IRIS Studio - 수원시 공공 데이터를 map 에 표시 
================================================================================
    
|

-----------------
요약 
-----------------

| 지도에 위/경도가 있는 데이터를 마커(점 또는 깃발)로 찍어서 보여주려고 한다.
| 아래 처럼 5개의 아이템을 5개의 색깔로 점을 찍어서 전반적인 분포를 본다.
|
|     - 수원주차장표준데이터		: SUWON_PARKING_PLACE_INFORMATION
|     - 수원어린이보호구역표준데이터	: SUWON_CHILD_PROTECTION_AREA_INFORMATION
|     - 수원공공시설개방표준데이터	: SUWON_PUBLIC_FACILITY_OPENING_INFORMATION
|     - 수원CCTV표준데이터		: SUWON_CCTV_INFORMATION
|     - 수원보안등정보표준데이터	: SUWON_SECURITY_LAMP_INFORMATION

|

------------------
데이터 
------------------

|

- 모든 데이터는 IRIS DB 테이블에 있다는 가정이다.

- DB 브라우저에서 데이터 확인하기

.. image:: ../images/map_suwon/sw_1.png
    :height: 400
    :width: 800
    :scale: 100%
    :alt: DB브라우저에서 데이터 확인-1

|

- 위/경도 컬럼 확인 

.. image:: ../images/map_suwon/sw_2.png
    :height: 400
    :width: 800
    :scale: 100%
    :alt: DB브라우저에서 데이터 확인-2

|

--------------------
최종 지도 예시
--------------------

|

.. image:: ../images/map_suwon/last1.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: 수원시 데이터 지도

|
|

---------------------------------------------------------------
IRIS Studio : map 에서 여러개의 레이어로 아이템 표시하기 
---------------------------------------------------------------

'''''''''''''''''''''''''''''''''''''''''
새 보고서 생성하기  
'''''''''''''''''''''''''''''''''''''''''

- 보고서 메뉴에서 **새보고서** 클릭
    
.. image:: ../images/map_suwon/sw_4.png
    :height: 250
    :width: 800
    :scale: 100%
    :alt: 새보고서

|

- 텍스트 박스에 내용 추가하기
    
    - 텍스트 박스에 내용을 추가하는 것은 오른쪽 **속성** 의 **기본값** 에 입력한다.

.. image:: ../images/map_suwon/sw_text01.png
    :alt: 새보고서


- 첫번째 layer map(지도) : open street map 선택
- 지도의 기본 위치로 **수원** 이 오도록 한 후 이 값으로 **현재 지도값으로 설정** 하기

.. image:: ../images/map_suwon/sw_map_layer.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: map layer

|

- 레이어 5개를 추가로 설정한다. 

    - 각각 보여주려는 아이템 이름으로 layer 이름을 정한다.(권장)

.. image:: ../images/map_suwon/sw_layer_add_1.png
    :height: 250
    :width: 700
    :alt: map layer add

|

- 아이템 선택을 위한 **체크 박스** 만들기 : 주차장, 어린이보호구역, 공공시설개방, CCTV, 보안등정보
    
    - 한 개의 체크박스에 1개의 layer 를 선택하도록 총 5개의 체크박스를 따로 만든다.

.. image:: ../images/map_suwon/sw_chb_1.png
    :height: 220
    :width: 400
    :scale: 100%
    :alt: 체크박스_1

|

.. image:: ../images/map_suwon/sw_chb_2.png
    :height: 300
    :width: 600
    :scale: 100%
    :alt: 체크박스_2

|

- 수시로 **저장** !!!!

- 다시 **지도** 를 선택

- 주차장 layer 의 데이터를 가져오기 위한 설정값 입력한다.
  
    - IRIS DB 테이블에서 select 하는 SQL문을 오른쪽 **검색어** 에 입력한 후 **미리보기** 로 확인한다.

.. image:: ../images/map_suwon/sw_layer1_1.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: layer_1 data

|

- 주차장 layer 의 데이터는 주차장 체크박스 에서 선택되면 실행되도록 트리거 설정한다.


.. image:: ../images/map_suwon/sw_layer2_1.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: layer_1 ch


|


- 주차장 layer 의 시각화 설정하기
    - 시각화 유형은 위/경도 좌표를 마커(점) 으로 표시하기
    
.. image:: ../images/map_suwon/sw_layer3.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: layer_1 마커

|

- 마커의 시각화 옵션 설정하기
    - 마커의 종류 및 갯수, 마커의 크기 지정

.. image:: ../images/map_suwon/sw_layer_mk_size.png
    :alt: layer_1 마커 사이즈

|

- 마커의 색상 설정 : 주차장의 색상을 정하는 컬럼(여기서는 PARTITION_DATE 를 지정함)에 따라 그라디언트로 표현한다.
    - 임계치 및 객체별 자동은 데이터 및 case 에 따라 지정할 수 있으므로 사용자 메뉴얼을 참고할 것

.. image:: ../images/map_suwon/sw_layer_mk_color.png
    :alt: layer_1 마커 색상

|


- 마커의 데이터 설정 : 마커의 위/경도에 해당하는 컬럼을 지정한다.
    - 색상 컬럼은 group by 절의 컬럼 에 해당하며, 주차장 마커의 색상을 다르게 표현하고 샆을 때 사용한다.
    - 마커 색상 탭에서 그라디언트로 지정한 색상에 따라 주차장 마커 색이 표현된다.
    - 여기서는 모두 동일한 날짜의 데이터이므로 주차장 마커의 색은 같은 색상이다.


.. image:: ../images/map_suwon/sw_layer_mk_data.png
    :scale: 100%
    :alt: layer_1 데이터

|

- 마커의 툴팁 설정 : 지도에서 특정 주차장 마커에 커서를 대면 보이는 정보를 설정한다.
    - 만약 컬럼이 보이지 않으면 **실행** 버튼을 눌러서 지도에 주차장 마커가 표시되게 한다.
    - 그 후에 마커의 시각화 옵션의 툴팁 설정 창을 열면 툴팁으로 보여 줄 수 있는 컬럼이 보인다.
    - 이 컬럼은 지도의 데이터 항목에서 IRIS DB 에 보낸 SQL구문의 컬럼들이다.

.. code::

    /*+ LOCATION ( PARTITION = '20191017000000' ) */ 
    SELECT 
	    PARTITION_DATE, 
        PARKING_PLACE_NAME as FACILITY_NAME, 
        PARKING_PLACE_MANAGEMENT_NUMBER,
        PARKING_PLACE_SECTION, PARKING_PLACE_TYPE,
        PLACE_OF_LOCATION_ROAD_NAME_ADDRESS as ADDRESS,  
        PARKING_COMPARTMENT_COUNT, OPERATION_DAY,
        WEEKDAY_OPERATION_BEGIN_TIME, WEEKDAY_OPERATION_END_TIME, 
        SATURDAY_OPERATION_BEGIN_TIME, SATURDAY_OPERATION_END_TIME, 
        HOLIDAY_OPERATION_BEGIN_TIME, HOLIDAY_OPERATION_END_TIME, 
        CHARGE_INFORMATION, PARKING_BASIS_TIME, PARKING_BASIS_CHARGE, 
        ADDITION_UNIT_TIME, ADDITION_UNIT_CHARGE, DAY_PARKING_TICKET_CHARGE_APPLICATION_TIME, 
        DAY_PARKING_TICKET_CHARGE, MONTH_FIXED_TERM_TICKET_CHARGE, PAY_METHOD, SPECIAL_MATTER, 
        MANAGEMENT_INSTITUTION_NAME, TELEPHONE_NUMBER,
        LATITUDE, LONGITUDE
    FROM 
	    JPHONG.SUWON_PARKING_PLACE_INFORMATION
    ;



.. image:: ../images/map_suwon/sw_layer_mk_tt.png
    :scale: 100%
    :alt: layer_1 마커 툴팁

|

- 툴팁 실행 예시

.. image:: ../images/map_suwon/sw_layer_mk_tt_2.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: layer_1 툴팁 예시


- 동일한 방법으로 나머지 어린이보호구역/공공시설개방/CCTV/보안등정보 레이어를 생성한다.

|
|

- 각 레이어의 마커 색상 정보를 보기 쉽게 하기 위해 **범례** 를 따로 만들기로 한다.

.. image:: ../images/map_suwon/desc1.png
    :height: 50
    :width: 300
    :scale: 100%
    :alt: 범례

- 주차장 레이어의 마커 색상 정보를 복사한다.

.. image:: ../images/map_suwon/desc2.png
    :height: 100
    :width: 250
    :scale: 100%
    :alt: layer_1 마커

- 메뉴바에서 **텍스트상자** 클릭

.. image:: ../images/map_suwon/desc3.png
    :height: 20
    :width: 200
    :scale: 100%
    :alt: 텍스트상자

- 텍스트 상자를 지도 위에 적당한 크기로 그리고, 속성탭에서 기본값으로 주차장 입력한다.

.. image:: ../images/map_suwon/parking_att.png
    :height: 150
    :width: 300
    :scale: 100%
    :alt: 주차장범례 속성

- 메뉴바에서 사각형 을 선택하고, 주차장 텍스트 박스 아래에 두고 복사한 주차장 마커의 색상 정보를 설정한다.

.. image:: ../images/map_suwon/polygon4_att.png
    :height: 150
    :width: 300
    :scale: 100%
    :alt: 주차장범례 속성

- 다른 레이어의 범례도 같은 방법으로 생성한다.

- 최종 보기

.. image:: ../images/map_suwon/sw_last.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: 최종

|

- 참고로 현재 체크박스에서 선택을 삭제해도 지도에서는 마커가 그대로 보이므로, 
   
    - re-load 하여 다시 체크박스에서 선택하거나
   
    - 지도의 레이어팝업 창에서 레이어별로 보기를 선택하는 방법을 사용해야 한다.

