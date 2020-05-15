서빙 명령어 활용 - boston housing 데이터 예제
====================================================================================================

boston housing 데이터로 집값 예측 예제 입니다.

학습
----------------------------------------------------------------------------------------------------

| 13개의 집값 관련 속성으로 이루어진 404 레코드를 다운로드, 전처리, 학습합니다.
| 데이터에 대한 설명은 `여기 <https://keras.io/datasets/#usage_6>`_ 를 참조해주세요. 

tensorflow 2.1.0 패키지가 필요합니다.

``pip install tensorflow==2.1.0``

.. code-block:: none

   import numpy as np # linear algebra
   
   from keras.datasets import boston_housing
   from tensorflow import keras
   from tensorflow.keras import layers
   
   # 다운로드
   (train_data, train_targets), (test_data, test_targets) = boston_housing.load_data()
   
   # 전처리
   mean = train_data.mean(axis=0)
   train_data -= mean
   std = train_data.std(axis=0)
   train_data /= std
   test_data -= mean
   test_data /= std

   # 학습 모델
   def build_model():
       model = keras.Sequential([
       layers.Dense(64, activation='relu', input_shape=(train_data.shape[1],)),
       layers.Dense(64, activation='relu'),
       layers.Dense(1)
       ])
       model.compile(optimizer='rmsprop',
                 loss='mse',
                 metrics=['mae'])
       return model
   
   model = build_model()
   # 학습
   model.fit(train_data, train_targets, epochs=80, batch_size=16, verbose=0)
   # 평가
   test_mse_score, test_mae_score = model.evaluate(test_data, test_targets)
   

개인 객체저장소에 모델 업로드
----------------------------------------------------------------------------------------------------

| IRIS Discovery Service에 적재하기 위해, tar 파일로 압축하여 개인 객체저장소에 업로드합니다.
| boto3 패키지가 필요합니다.

``pip install boto3``

- 아래 인자를 입력해주세요.

 - bucket : 개인 객체 저장소의 bucket
 - key : 개인 객체 저장소의 key
 - endpoint_url : 개인 객체 저장소의 url
 - aws_access_key_id : 개인 객체 저장소의 access_key
 - aws_secret_access_key : 개인 객체 저장소의 secret_access_key

.. code-block:: none

   import tempfile
   import tarfile
   import os
   import boto3
   import tensorflow as tf
   
   # 개인 객체저장소 설정
   bucket = 'iris'
   key = 'boston_housing/model.tar'
   setting = {
       'endpoint_url': "http://192.168.102.138:9003",
       'verify': False,
       'aws_access_key_id': 'minio',
       'aws_secret_access_key': 'minio123'
   }

   # 모델 생성
   export_path = tempfile.mkdtemp()
   tf.keras.models.save_model(
       model,
       export_path,
       overwrite=True,
       include_optimizer=True,
       save_format=None,
       signatures=None,
       options=None
   )

   # 모델 압축
   tar_name = export_path + '/model.tar'
   with tarfile.open(tar_name, "w:tar") as tar:
       tar.add(export_path, arcname='./')

   # 모델 업로드
   cli = boto3.client('s3', **setting)
   cli.upload_file(tar_name, bucket, key)


적재
----------------------------------------------------------------------------------------------------   

| IRIS Discovery Service에 모델을 적재합니다.
| 적재는 IRIS Discovery Service의 `mlmodel import  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/mlmodel.html#mlmodel-import>`_ 를 사용합니다.

IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다. path 옵션에 개인 객체저장소 정보, tar로 압축한 모델 경로를 입력합니다.

``mlmodel import name=house type=tf category=classification algorithm=deep format=saved_model path=OBJECTSTORAGE.{CONNECTOR NAME}:boston_housing/model.tar``

결과

.. list-table::
   :header-rows: 1

   * - result
   * - ok


배포
----------------------------------------------------------------------------------------------------   

| IRIS Discovery Service가 관리하는 tensorflow serving에 모델을 배포합니다.
| 배포는 IRIS Discovery Service의 `mlmodel deploy  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/mlmodel.html#mlmodel-deploy>`_ 를 사용합니다.

IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다.

``mlmodel deploy house label='first house'``

결과

- house이 root_house 이름으로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - result
     - latest_version
     - serving_name
   * - ok
     - 1
     - root_house

서빙 상태 확인
----------------------------------------------------------------------------------------------------        

| 배포한 multi_in_out모델의 서빙 상태를 확인합니다.
| 서빙 상태 확인은 IRIS Discovery Service의 `serving status  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/serving.html#serving-status>`_ 를 사용합니다.

IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다.

``serving status house``

결과

- multi_in_out 모델로 생성한 version 1이 사용 가능한 상태로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - version
     - state
     - label
   * - 1
     - AVAILABLE
     - first house

테스트 데이터 업로드
----------------------------------------------------------------------------------------------------        

아래 스크립트를 실행하여, 로컬에 boston_housing_test.csv파일 생성 후, 개인 객체 저장소에 업로드합니다.

.. code-block:: none
   
   from keras.datasets import boston_housing
   import numpy as np
   
   # 다운로드
   (train_data, train_targets), (test_data, test_targets) = boston_housing.load_data()
   
   # 전처리
   mean = train_data.mean(axis=0)
   train_data -= mean
   std = train_data.std(axis=0)
   train_data /= std
   test_data -= mean
   test_data /= std
   
   # 로컬에 저장
   filename = 'boston_housing_test.csv'
   save_data = []
   for t in test_data:
       save_data.append(','.join([str(x) for x in t]))
   new_test_data = np.array(save_data)
   np.savetxt(filename, new_test_data, fmt="%s", delimiter=",", header="feature", comments='')

`연결 정보 생성 가이드 <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Common/inquiry_management/connect_info/index.html#id4>`_ 를 참고하여, 개인 객체 저장소를 연결 정보에 추가합니다.

`데이터 모델 생성 가이드 <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Analyzer/data_model/00_data_model.html#id6>`_ 를 참고하여, ``boston_housing_test`` 이름으로 데이터 모델을 생성합니다.

예측
----------------------------------------------------------------------------------------------------        

배포된 모델에 대해 4가지 유형의 예측 방법이 있습니다.

- python 스크립트 방식
- DSL 설정파일 방식
- DSL 데이터 소스 입력 방식
- curl 방식

| 이중 DSL 데이터 소스 입력 방식에 대해 진행합니다. 
| python 스크립트 방식, DSL 설정파일 방식, curl 방식은 다음 유즈케이스를 참조해주세요.
`mnist 옷 모델 적재, 예측  <http://docs.iris.tools/manual/IRIS-Usecase/ml-serving/mnist_clothes.html>`_ 을 참조해주세요.


DSL 데이터 소스 입력 방식
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 데이터 소스를 입력하여 예측합니다.
| 예측(서빙)은 IRIS Discovery Service의 `serving predict  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/serving.html#serving-predict>`_ 를 사용합니다.

IRIS Discovery Service에서 boston_housing_test 모델 선택 후, 검색창에 아래 명령어를 입력합니다.

``* | serving predict house feature=[(feature,dense_input,float,13)]``

결과

.. list-table::
   :header-rows: 1

   * - feature
     - predictions
   * - 1.5536935453162368,-0.4836154708652843,1.02832...
     - [8.03]
   * - 0.39242675047976094,-0.4836154708652843,-0.16...
     - [17.92]
   * - -0.2678050396682621,-0.4836154708652843,1.2458...
     - [32.57]
   * - ...
     - ...