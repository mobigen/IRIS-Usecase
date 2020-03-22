================================================================================
IRIS Studio - 수원시 공공 데이터 예제
================================================================================
    


위/경도 좌표가 있는 데이터를 마커(점 또는 깃발)로 찍어서 지도로 보여주는 예제입니다.
수원시의 공공 데이터 5개를 각각의 레이어로 만들어서 지도에 포인트로 표시합니다.

  - 수원주차장표준데이터		 : SUWON_PARKING_PLACE_INFORMATION
  - 수원어린이보호구역표준데이터   : SUWON_CHILD_PROTECTION_AREA_INFORMATION
  - 수원공공시설개방표준데이터     : SUWON_PUBLIC_FACILITY_OPENING_INFORMATION
  - 수원CCTV표준데이터		   : SUWON_CCTV_INFORMATION
  - 수원보안등정보표준데이터	  : SUWON_SECURITY_LAMP_INFORMATION


.. contents::
    :backlinks: top




--------------------
최종 지도 예시
--------------------

.. image:: ../images/map_suwon/sw_map01.png
    :alt: 수원시 데이터 지도



---------------------------------------------------------------
IRIS Studio : map 에서 여러개의 레이어로 아이템 표시하기 
---------------------------------------------------------------

'''''''''''''''''''''''''''''''''''''''''
새 보고서 생성하기  
'''''''''''''''''''''''''''''''''''''''''

- 보고서 메뉴에서 **새보고서** 클릭합니다.
    
.. image:: ../images/map_suwon/sw_4.png
    :alt: 새보고서



- 텍스트 박스에 내용 및 설명을 추가합니다.
    - 텍스트 박스에 내용을 추가하는 것은 오른쪽 **속성** 의 **기본값** 에 입력합니다.

.. image:: ../images/map_suwon/sw_map03.png
    :alt: 새보고서



'''''''''''''''''''''''''
수원시 지도 설정
'''''''''''''''''''''''''

- 첫번째 layer map(지도) : open street map 선택합니다.
- 지도의 기본 위치로 **수원** 이 오도록 한 후 이 값으로 **현재 지도값으로 설정**  합니다.

.. image:: ../images/map_suwon/sw_map_layer.png
    :height: 450
    :width: 800
    :scale: 100%
    :alt: map layer



''''''''''''''''''''''''''
레이어 추가하기
''''''''''''''''''''''''''

- 레이어 5개를 추가로 설정합니다.
    - 각각 보여주려는 아이템 이름으로 layer 이름을 정하는 것을 권장합니다.

.. image:: ../images/map_suwon/sw_layer_add_1.png
    :alt: map layer add



''''''''''''''''''''''''''''''''''
주차장 레이어 만들기
''''''''''''''''''''''''''''''''''

- 다시 **지도** 를 선택합니다.

- 주차장 layer 의 데이터를 가져오기 위한 설정값을 입력합니다.
     - IRIS DB 테이블에서 select 하는 SQL문을  **검색어** 에 입력한 후 **미리보기** 로 확인해 봅니다.
     - 주차장 layer 의 데이터를 가져오는 방법은 **데이터실행방법설정** 에서 자동 실행으로 설정해야 합니다.
     - 실행 버튼을 누릅니다.

.. image:: ../images/map_suwon/sw_map02.png
    :scale: 60%
    :alt: layer_1 data


''''''''''''''''''''''''''''''''''
주차장 마커 시각화 설정
''''''''''''''''''''''''''''''''''

- 주차장 layer 의 시각화 부분을 설정합니다.
    - 시각화 유형은 위/경도 좌표를 마커(점) 으로 표시합니다.
    
.. image:: ../images/map_suwon/sw_map04.png
    :scale: 60%
    :alt: layer_1 마커


- 마커의 시각화 옵션을 설정하는 방법입니다.
    - 마커의 종류 및 갯수, 마커의 크기 지정할 수 있습니다.

.. image:: ../images/map_suwon/sw_map05.png
    :scale: 60%
    :alt: layer_1 마커 사이즈


- 마커의 색상 설정 : 주차장의 색상을 정하는 컬럼(여기서는 PARTITION_DATE)에 따라 그라디언트로 표현합니다.
    - 임계치 및 객체별 자동은 데이터 및 case 에 따라 지정할 수 있으므로 사용자 메뉴얼을 참고하세요.

.. image:: ../images/map_suwon/sw_layer_mk_color.png
    :scale: 60%
    :alt: layer_1 마커 색상


- 마커의 데이터 설정 : 마커의 위/경도에 해당하는 컬럼을 지정합니다.
    - 색상 컬럼은 group by 절의 컬럼 에 해당하며, 주차장 마커의 색상을 다르게 표현하고 샆을 때 사용합니다.
    - 마커 색상 탭에서 그라디언트로 지정한 색상에 따라 주차장 마커 색이 표현됩니다.
    - 여기서는 모두 동일한 날짜의 데이터이므로 주차장 마커의 색은 같은 색상이 됩니다.

.. image:: ../images/map_suwon/sw_layer_mk_data.png
    :scale: 60%
    :alt: layer_1 데이터



'''''''''''''
툴팁
'''''''''''''

- 마커의 툴팁 설정 : 지도에서 특정 주차장 마커에 커서를 대면 보여지는 내용(툴팁)을 지정하는 부분입니다.
    - 만약 컬럼이 보이지 않으면 **실행** 버튼을 눌러서 지도에 주차장 마커가 표시되게 합니다.
    - 그 후에 마커의 시각화 옵션의 툴팁 설정 창을 열면 툴팁으로 보여 줄 수 있는 컬럼이 보여집니다.
    - 이 컬럼은 지도의 데이터 항목에서 IRIS DB 에 보낸 SQL구문의 컬럼들입니다.

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
    :scale: 60%
    :alt: layer_1 마커 툴팁



- 툴팁 실행 예시

.. image:: ../images/map_suwon/sw_layer_mk_tt_2.png
    :alt: layer_1 툴팁 예시


- 동일한 방법으로 나머지 어린이보호구역/공공시설개방/CCTV/보안등정보 레이어를 생성할 수 있습니다.


''''''''''''''''''''''''''''''''''
지도에서 레이어 선택 방법
''''''''''''''''''''''''''''''''''

- 레이어 선택은 지도의 레이어 버튼으로 지도에 표시 여부를 지정할 수 있습니다.

.. image:: ../images/map_suwon/sw_map06.png
    :alt: 레이어 선택



''''''''''''''''''''
범례 만들기
''''''''''''''''''''

- 각 레이어의 마커 색상 정보를 보기 쉽게 하기 위해 **범례** 는 따로 만들어 봅니다.

.. image:: ../images/map_suwon/desc1.png
    :scale: 60%
    :alt: 범례

- 주차장 레이어의 마커 색상 정보를 복사합니다.

.. image:: ../images/map_suwon/desc2.png
    :scale: 60%
    :alt: layer_1 마커

- 메뉴바에서 **텍스트상자** 클릭합니다.

.. image:: ../images/map_suwon/desc3.png
    :scale: 60%
    :alt: 텍스트상자

- 텍스트 상자를 지도 위에 적당한 크기로 그리고, 속성탭에서 기본값으로 주차장 입력합니다.

.. image:: ../images/map_suwon/parking_att.png
    :scale: 60%
    :alt: 주차장범례 속성

- 메뉴바에서 사각형 을 선택하고, 주차장 텍스트 박스 아래에 두고 복사한 주차장 마커의 색상 정보를 설정합니다.

.. image:: ../images/map_suwon/polygon4_att.png
    :scale: 60%
    :alt: 주차장범례 속성

- 다른 레이어의 범례도 같은 방법으로 생성합니다.


''''''''''''''''''''''''''
최종  지도 
''''''''''''''''''''''''''

.. image:: ../images/map_suwon/sw_map01.png
    :alt: 최종



