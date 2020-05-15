ML 명령어 - multi input multi output 예제
====================================================================================================

랜덤데이터로 학습을 진행하고, 개인 객체 저장소에 업로드하여 IRIS Discovery Service 적재, 서빙하는 시나리오입니다.

학습
----------------------------------------------------------------------------------------------------

| 입력을 10개의 input으로 이루어진 title, body, 12개의 input으로 이루어진 tag 로,
| 출력을 1개의 output을 갖는 priority, 4개의 output을 갖는 department 로 학습합니다.

`여기 <https://www.tensorflow.org/guide/keras/functional#manipulate_complex_graph_topologies>`_ 를 참조하였습니다.

tensorflow 2.1.0 패키지가 필요합니다.

``pip install tensorflow==2.1.0``

.. code-block:: none

   from tensorflow import keras
   from tensorflow.keras import layers
   import numpy as np
   
   num_tags = 12  # Number of unique issue tags
   num_words = 10000  # Size of vocabulary obtained when preprocessing text data
   num_departments = 4  # Number of departments for predictions
   
   title_input = keras.Input(shape=(None,), name='title')#, dtype='string')  # Variable-length sequence of ints
   body_input = keras.Input(shape=(None,), name='body')  # Variable-length sequence of ints
   tags_input = keras.Input(shape=(num_tags,), name='tags')  # Binary vectors of size `num_tags`
   
   # Embed each word in the title into a 64-dimensional vector
   title_features = layers.Embedding(num_words, 64)(title_input)
   # Embed each word in the text into a 64-dimensional vector
   body_features = layers.Embedding(num_words, 64)(body_input)
   
   # Reduce sequence of embedded words in the title into a single 128-dimensional vector
   title_features = layers.LSTM(128)(title_features)
   # Reduce sequence of embedded words in the body into a single 32-dimensional vector
   body_features = layers.LSTM(32)(body_features)
   
   # Merge all available features into a single large vector via concatenation
   x = layers.concatenate([title_features, body_features, tags_input])
   
   # Stick a logistic regression for priority prediction on top of the features
   priority_pred = layers.Dense(1, name='priority')(x)
   # Stick a department classifier on top of the features
   department_pred = layers.Dense(num_departments, name='department')(x)
   
   # Instantiate an end-to-end model predicting both priority and department
   model = keras.Model(inputs=[title_input, body_input, tags_input],
                       outputs=[priority_pred, department_pred])
   
   model.compile(optimizer=keras.optimizers.RMSprop(1e-3),
                 loss=[keras.losses.BinaryCrossentropy(from_logits=True),
                       keras.losses.CategoricalCrossentropy(from_logits=True)],
                 loss_weights=[1., 0.2])
   
   # Dummy input data
   title_data = np.random.randint(num_words, size=(1280, 10))
   body_data = np.random.randint(num_words, size=(1280, 100))
   tags_data = np.random.randint(2, size=(1280, num_tags)).astype('float32')
   
   # Dummy target data
   priority_targets = np.random.random(size=(1280, 1))
   dept_targets = np.random.randint(2, size=(1280, num_departments))
   
   model.fit({'title': title_data, 'body': body_data, 'tags': tags_data},
             {'priority': priority_targets, 'department': dept_targets},
             epochs=2,
             batch_size=32)
   

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
   key = 'multi/model.tar'
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

``mlmodel import name=multi_in_out type=tf category=classification algorithm=deep format=saved_model path=OBJECTSTORAGE.{CONNECTOR NAME}:multi/model.tar``

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

``mlmodel deploy multi_in_out label='first multi'``

결과

- multi_in_out이 root_multi_in_out 이름으로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - result
     - latest_version
     - serving_name
   * - ok
     - 1
     - root_multi_in_out

서빙 상태 확인
----------------------------------------------------------------------------------------------------        

| 배포한 multi_in_out모델의 서빙 상태를 확인합니다.
| 서빙 상태 확인은 IRIS Discovery Service의 `serving status  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/serving.html#serving-status>`_ 를 사용합니다.

IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다.

``serving status multi_in_out``

결과

- multi_in_out 모델로 생성한 version 1이 사용 가능한 상태로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - version
     - state
     - label
   * - 1
     - AVAILABLE
     - first multi

테스트 데이터 업로드
----------------------------------------------------------------------------------------------------        

아래 데이터 내용을 multi_in_out.tsv 파일명으로 생성 후, 개인 객체 저장소에 업로드합니다.

.. code-block:: none
   
   title	body	tags
   [0.43, 0.77, 0.3, 0.19, 0.38, 0.37, 0.56, 0.48, 0.8, 0.4]	[0.9, 0.5, 0.16, 0.74, 0.9, 0.64, 0.37, 0.18, 0.08, 0.87]	[0.44, 0.45, 0.56, 0.63, 0.72, 0.28, 0.57, 0.19, 0.66, 0.47, 0.89, 0.37]
   [0.64, 0.14, 0.6, 0.02, 0.56, 0.77, 0.5, 0.33, 0.33, 0.83]	[0.49, 0.97, 0.28, 1.0, 0.03, 0.97, 0.96, 0.6, 0.75, 0.01]	[0.65, 0.89, 0.55, 0.06, 0.31, 0.38, 0.78, 0.45, 0.56, 0.1, 0.58, 0.79]
   [0.73, 0.33, 0.58, 0.28, 0.15, 0.98, 0.46, 0.56, 0.39, 0.95]	[0.0, 0.45, 0.27, 0.22, 0.75, 0.05, 0.14, 0.45, 0.35, 0.87]	[0.79, 0.24, 0.54, 0.93, 0.89, 0.73, 0.58, 0.18, 0.03, 0.92, 0.54, 0.37]
   [0.18, 0.21, 0.28, 0.62, 0.21, 0.3, 0.54, 0.05, 0.54, 0.09]	[0.06, 0.01, 0.01, 0.5, 0.74, 0.17, 0.68, 0.56, 0.3, 0.12]	[0.81, 0.31, 0.82, 0.16, 0.95, 0.39, 0.88, 0.15, 1.0, 0.41, 0.4, 0.43]
   [0.63, 0.9, 0.56, 0.81, 0.08, 0.67, 0.22, 0.82, 0.06, 0.68]	[0.91, 0.2, 0.71, 0.39, 0.5, 0.5, 0.36, 0.81, 0.46, 0.46]	[0.25, 0.61, 0.27, 0.3, 0.64, 0.02, 0.31, 0.77, 0.54, 0.01, 0.27, 0.39]

`연결 정보 생성 가이드 <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Common/inquiry_management/connect_info/index.html#id4>`_ 를 참고하여, 개인 객체 저장소를 연결 정보에 추가합니다.

`데이터 모델 생성 가이드 <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Analyzer/data_model/00_data_model.html#id6>`_ 를 참고하여, ``multi_in_out`` 이름으로 데이터 모델을 생성합니다.

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

IRIS Discovery Service에서 multi_in_out 모델 선택 후, 검색창에 아래 명령어를 입력합니다.

``* | serving predict multi_in_out feature=[(title, title, float, 10), (body, body, float, 10), (tags, tags, float, 12)]``

결과

.. list-table::
   :header-rows: 1

   * - title
     - body
     - tags
     - priority
     - department
   * - [0.43, 0.77, 0.3, 0.19, 0.38, 0.37, 0.56, 0.48...
     - [0.9, 0.5, 0.16, 0.74, 0.9, 0.64, 0.37, 0.18, ...
     - [0.44, 0.45, 0.56, 0.63, 0.72, 0.28, 0.57, 0.1...
     - [0.432420015]
     - [16.4958725, 17.1233311, 17.7156048, 17.6724377]
   * - [0.64, 0.14, 0.6, 0.02, 0.56, 0.77, 0.5, 0.33,...
     - [0.49, 0.97, 0.28, 1.0, 0.03, 0.97, 0.96, 0.6,...
     - [0.65, 0.89, 0.55, 0.06, 0.31, 0.38, 0.78, 0.4...
     - [0.258726358]
     - [16.4389248, 17.0613327, 17.5822201, 17.6518764]
   * - ...
     - ...
     - ...
     - ...
     - ...