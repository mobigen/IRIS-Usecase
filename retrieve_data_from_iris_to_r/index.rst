R 에서 아이리스에 저장된 데이터 가져오기
=============================================================================


-----------------
목차
-----------------

- 설명 

- RJDBC 를 이용하여 IRIS DB 접속하기

- IRIS Global 테이블 생성하기

- IRIS Global 테이블에 데이터를 insert / select 하기

- IRIS Local 테이블 생성하기

- IRIS Local 테이블에 데이터를 insert / select 하기

|
-----------------
설명 
-----------------

R-Studio 에서 RJDBC 패키지를 이용하여 IRIS DB 에 테이블을 create 하고, 데이터를 insert, select 하는 예제입니다.

IRIS 의 global 테이블과 local 테이블을 특성에 맞게 각각 생성해 보고 데이터를 insert / select 해 봅니다.

|
|


-----------------------------------------------------
RJDBC 를 이용하여 IRIS DB 접속하기
-----------------------------------------------------

- RJDBC 패키지를 이용합니다. 
    - id / passwd = myiris / myiris 
    - iris DB 접속 정보 : 192.168.100.180:5050
    - JDBC path 지정

.. code::

  library(RJDBC)

  .jinit()
  .jaddClassPath("/usr/share/R/library/jettison-1.3.2.jar")
  .jaddClassPath("/usr/share/R/library/log4j-1.2.17.jar")
  .jaddClassPath("/docker/tools/Spark-on-IRIS/lib/java/mobigen-iris-jdbc-2.1.0.1.jar")

 

|

- java class path 확인 

.. code::

  print(.jclassPath())
 
  [1] "/usr/lib64/R/library/rJava/java"                                   
  [2] "/docker/tools/Spark-on-IRIS/lib/java/mobigen-iris-jdbc-2.1.0.1.jar"
  [3] "/usr/lib64/R/library/RJDBC/java/RJDBC.jar"

|

- IRIS DB 접속 connector object( conn )  생성하는 코드입니다.
    - myiris 계정 예제
    - jdbc driver 버전  : mobigen-iris-jdbc-2.1.0.1.jar

.. code::

  drv <- RJDBC::JDBC("com.mobigen.iris.jdbc.IRISDriver",
                     "/docker/tools/Spark-on-IRIS/lib/java/mobigen-iris-jdbc-2.1.0.1.jar", 
                      identifier.quote= "`")
  conn <- RJDBC::dbConnect(drv, "jdbc:iris://192.168.100.180:5050/myiris", "myiris", "myiris")


|


----------------------------------------------
IRIS Global 테이블 생성하기
----------------------------------------------

- 붓꽃(iris) 데이터를 기준으로 한 Global 테이블을 생성해 봅니다.

- Global 테이블 생성 SQL 

.. code::

  sql_create <-  "CREATE TABLE IRIS_GLOBAL_TEST_1 (
                  irisid INTEGER, 
                  sepal_length REAL,
                  sepal_width REAL,
                  petal_length REAL,
                  petal_width REAL,
                  species TEXT)
                  datascope       GLOBAL
                  ramexpire       0
                  diskexpire      0
                  partitionkey    None
                  partitiondate   None
                  partitionrange  0 ; "

|

- dbSendUpdate 로 sql 문을 실행합니다.
    - return 되는 값은 없으니 주의하세요!!!!

.. code::

  # 테이블이 있다면 DROP 하고 생성(drop 문은 항상 주의해 주세요!!) 합니다.
  dbSendUpdate(conn, 'DROP TABLE IF EXISTS MYIRIS.IRIS_GLOBAL_TEST_1 ;') 


  # CREATE GLOBAL TABLE : dbSendUpdate 를 이용하며, SQL 문 끝에 ; 를 꼭 넣어야 합니다. 
  # return값은 없습니다.
  dbSendUpdate(conn, sql_create) 

|

- 테이블이 생성되었는 지 확인합니다.
    - IRIS DB 명령어인 "table list" 사용합니다.

.. code::

  table_list <- dbGetQuery(conn, "table list") 
  print(table_list)


|

-------------------------------------------------------------------
IRIS Global 테이블에 Insert / select
-------------------------------------------------------------------

- Insert into Global table
    - INSERT INTO 테이블명 VALUES (,,,,) 예제
    - dbSendUpdate

.. code::

  ins_sql <- sprintf( "INSERT INTO IRIS_GLOBAL_TEST_1 (irisid, sepal_length,sepal_width, petal_length,petal_width,  species) VALUES (1, 1.0, 2.0, 3.0, 4.0, 'test') ; ")

  dbSendUpdate(conn, ins_sql)

|

- Select from Global table
    - dbGetQuery

.. code::

  # SELECT from GLOBAL TABLE
  result2 <- dbGetQuery(conn, 'select * from IRIS_GLOBAL_TEST_1')
  print(result2)


|
|

--------------------------------------------
IRIS Local 테이블 생성하기
--------------------------------------------

- SYSLOG 데이터를 기준으로 Local 테이블을 생성합니다.
    - partiton range = 60min ( 60분으로 파티션 범위를 정합니다.)
    - partition 구분 컬럼 = DATETIME ( partition 구분기준 컬럼이름. 반드시 해당 시간필드를  YYYYMMDDHHMMSS 14자리 text 형식으로 변환해 놓아야 합니다)
    - partition키 = HOST ( partition key)

.. code::

  cr_table_sql <- 'CREATE TABLE IRIS_LOCAL_TEST_2 (
     DATETIME     TEXT,
     HOST         TEXT,
     FACILITY     TEXT,
     PRIORITY     TEXT,
     LEVEL        TEXT,
     LEVEL_INT    TEXT,
     TAG          TEXT,
     PROGRAM      TEXT )
  datascope       LOCAL
  ramexpire       60
  diskexpire      2102400
  partitionkey    HOST
  partitiondate   DATETIME
  partitionrange  60
  ; '

  dbSendUpdate(conn, cr_table_sql)

|

--------------------------------------------------------------------
IRIS Local 테이블로부터 데이터 Select  
--------------------------------------------------------------------

- IRIS DB 에 있는 Local 테이블로부터 데이터를 select 합니다.
- 가져온 데이터를 R dataframe 에 저장한 후 1000 개 단위로 새로 만든 테이블에 Insert 를 실행합니다.
    - dbSendQuery 로 데이터 select 
    - dbFetch 를 반복 해서 데이터를 가져 와서 dataframe 에 저장합니다.
    - dbClearResult 
    

.. code::

  # SELECT from LOCAL TABLE EVA.SYSLOG  
  # 2019-11-12 15:00:00 ~ 16:59:59 ( 20191112150000, 20191112160000 2개의 파티션에서 데이터를 가지고 옵니다)
  # count = 89887

  select_sql <- "/*+ LOCATION ( PARTITION >= '20191112150000' AND PARTITION <= '20191112160000' ) */ 
  SELECT 
  	  DATETIME, HOST, FACILITY, PRIORITY, LEVEL, LEVEL_INT, TAG, PROGRAM 
  FROM 
	  EVA.SYSLOG
  ; "

  my_dataframe <- data.frame()
  rs <- dbSendQuery(conn, select_sql)  # dbSendQuery !!!! ( dbGetQuery 아님 )

  nn = 1000   #  1000 개 단위로 fetch
  tmp_df <- data.frame()

  tmp_df <- dbFetch(rs, n=nn)
  my_dataframe <- tmp_df

  while ( nrow(tmp_df) ==  nn ) {
    tmp_df <- dbFetch(rs, n=nn)
    my_dataframe <- rbind(my_dataframe, tmp_df)
  }  
  dbClearResult(rs)   
  # TRUE
  print(nrow(tmp_df))   # 마지막 fetch 레코드 수 < nn 
  # 887


  # select 한 전체 레코드 수
  print(nrow(my_dataframe))
  # 89887


|


- R DataFrame 을 IRIS DB 에 Insert 하기
    - my_dataframe( 총 89887 건 ) :  1000 건을 한번에 insert 하는 예제입니다.
    - batch insert SQL 문을 만드는 function 생성 : insert_batch_sql_f
    - dbGetQuery  로 insert한 데이터를 확인합니다.

.. code::

  library(dplyr)

  table_name <- 'MYIRIS.IRIS_LOCAL_TEST_2'

  insert_batch_sql_f <- function(conn, table, df) {
    batch <- apply(df, 1, FUN = function(x) paste0("'",trimws(x),"'",collapse = ",")) %>% paste0("(",.,")",collapse = ", ")
 
    colums <-  paste(unlist(colnames(df)), collapse=',')
    query <- paste("INSERT INTO ", table, "(", colums, ") VALUES ", batch, ';')
  
    dbSendUpdate(conn, query)
  }

  insert_batch_sql_f(conn, table_name, my_dataframe[1:1000, ])
  # 1건씩 인서트는 인서트 sql 을 만들어서 dbSendUpdate(conn, query) 

  my_count <- dbGetQuery(conn,"select count(*) from IRIS_LOCAL_TEST_2")
  print(my_count)

|

-----------------------------------
R rmd  실행 결과 (PDF)
-----------------------------------


`Rmd 실행 결과 PDF <https://github.com/mobigen/IRIS-Usecase/blob/master/retrieve_data_from_iris_to_r/RJDBC_v0.2.utf8.md.pdf>`_