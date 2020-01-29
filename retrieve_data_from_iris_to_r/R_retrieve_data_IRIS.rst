.. sectnum::

================================================================================
R-studio : Retrieve data from IRIS DB
================================================================================
    
|

-----------------
요약 
-----------------

R-Studio 에서 RJDBC 를 이용하여 IRIS DB 에 테이블을 create 하고, select 하는 예제이다.

global 테이블과 local 테이블을 각각 create / select 해 본다.

|
|

-----------------
목차
-----------------

- RJDBC 를 이용하여 IRIS DB 접속하기

- IRIS Global 테이블 생성하기

- IRIS Global 테이블에 데이터를 insert / select 하기

- IRIS Local 테이블 생성하기

- IRIS Local 테이블에 데이터를 insert / select 하기

|

-----------------------------------------------------
RJDBC 를 이용하여 IRIS DB 접속하기
-----------------------------------------------------

- RJDBC 패키지 이용한다.  
    - id / passwd = myiris / myiris 
    - iris DB 접속 정보 : 192.168.100.180:5050
    
.. code::

  library(RJDBC)

  .jinit()
  .jaddClassPath("/usr/share/R/library/jettison-1.3.2.jar")
  .jaddClassPath("/usr/share/R/library/log4j-1.2.17.jar")
  .jaddClassPath("/docker/tools/Spark-on-IRIS/lib/java/mobigen-iris-jdbc-2.1.0.1.jar")

  print(.jclassPath())
 
  drv <- RJDBC::JDBC("com.mobigen.iris.jdbc.IRISDriver",
                     "/docker/tools/Spark-on-IRIS/lib/java/mobigen-iris-jdbc-2.1.0.1.jar", 
                      identifier.quote= "`")
  conn <- RJDBC::dbConnect(drv, "jdbc:iris://192.168.100.180:5050/myiris", "myiris", "myiris")




|

----------------------------------------------
IRIS Global 테이블 생성하기
----------------------------------------------




-------------------------------------------------------------------
IRIS Global 테이블에 데이터 Insert / select
-------------------------------------------------------------------



--------------------------------------------
IRIS Local 테이블 생성하기
--------------------------------------------



--------------------------------------------------------------------
IRIS Local 테이블에 데이터 Insert / Select 
--------------------------------------------------------------------



