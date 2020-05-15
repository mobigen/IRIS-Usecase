ML 명령어 - mnist number 예제
====================================================================================================

tensorflow의 숫자 데이터로 학습을 진행하고, 개인 객체 저장소에 업로드하여 IRIS Discovery Service 적재, 서빙하는 시나리오입니다.

학습
----------------------------------------------------------------------------------------------------

| 10개의 숫자 카테고리로 이루어진 7만장의 흑백 숫자 이미지를 다운로드, 전처리, 학습합니다.
| tensorflow 2.1.0 패키지가 필요합니다.

``pip install tensorflow==2.1.0``

.. code-block:: none

   import tensorflow as tf
   from tensorflow import keras
   
   fashion_mnist = keras.datasets.mnist
   (train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()
   
   # scale the values to 0.0 to 1.0
   train_images = train_images / 255.0
   test_images = test_images / 255.0
   
   # reshape for feeding into the model
   train_images = train_images.reshape(train_images.shape[0], 28, 28, 1)
   test_images = test_images.reshape(test_images.shape[0], 28, 28, 1)
   
   class_names = ['zero', 'one', 'two', 'three', 'four',
                  'five', 'six', 'seven', 'eight', 'nine']
   
   model = keras.Sequential([
       keras.layers.Conv2D(input_shape=(28,28,1), filters=8, kernel_size=3, 
                           strides=2, activation='relu', name='Conv1'),
       keras.layers.Flatten(),
       keras.layers.Dense(10, activation=tf.nn.softmax, name='Softmax')
   ])
   model.summary()
   
   testing = False
   epochs = 5
   
   model.compile(optimizer='adam', 
                 loss='sparse_categorical_crossentropy',
                 metrics=['accuracy'])
   model.fit(train_images, train_labels, epochs=epochs)
   
   test_loss, test_acc = model.evaluate(test_images, test_labels)
   print('\nTest accuracy: {}'.format(test_acc))


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
   key = 'number/model.tar'
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
| IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다. path 옵션에 개인 객체저장소 정보, tar로 압축한 모델 경로를 입력합니다.

``mlmodel import name=mnist_number type=tf category=classification algorithm=deep format=saved_model path=OBJECTSTORAGE.{CONNECTOR NAME}:model.tar``

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

``mlmodel deploy mnist_number label='first number'``

결과

- mnist_number이 root_mnist_number 이름으로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - result
     - latest_version
     - serving_name
   * - ok
     - 1
     - root_mnist_number

서빙 상태 확인
----------------------------------------------------------------------------------------------------        

| 배포한 mnist_number모델의 서빙 상태를 확인합니다.
| 서빙 상태 확인은 IRIS Discovery Service의 `serving status  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/serving.html#serving-status>`_ 를 사용합니다.

IRIS Discovery Service의 검색창에 아래 명령어를 입력합니다.

``serving status mnist_number``

결과

- mnist_number모델로 생성한 version 1이 사용 가능한 상태로 배포되었습니다.

.. list-table::
   :header-rows: 1

   * - version
     - state
     - label
   * - 1
     - AVAILABLE
     - first number

테스트 데이터 업로드
----------------------------------------------------------------------------------------------------        

예측을 위한 테스트 데이터를 IRIS에 업로드합니다. 아래 예제를 참조해주세요.

- `mnist 이미지 다운로드  <http://docs.iris.tools/manual/IRIS-Usecase/ml/general-purpose.html#mnist>`_ 
- `개인 객체 저장소에 업로드  <http://docs.iris.tools/manual/IRIS-Usecase/ml/general-purpose.html#id1>`_ 
- `연결정보 등록  <http://docs.iris.tools/manual/IRIS-Usecase/ml/general-purpose.html#id2>`_ 
- `전처리  <http://docs.iris.tools/manual/IRIS-Usecase/ml/general-purpose.html#id5>`_ 

예측
----------------------------------------------------------------------------------------------------        

배포된 모델에 대해 4가지 유형의 예측 방법이 있습니다.

- python 스크립트 방식
- DSL 설정파일 방식
- DSL 데이터 소스 입력 방식
- curl 방식

이중 DSL 데이터 소스 입력 방식에 대해 진행합니다. 
python 스크립트 방식, DSL 설정파일 방식, curl 방식은 다음 유즈케이스를 참조해주세요.
`mnist 옷 모델 적재, 예측  <http://docs.iris.tools/manual/IRIS-Usecase/ml-serving/mnist_clothes.html>`_ 을 참조해주세요.

DSL 데이터 소스 입력 방식
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 데이터 소스를 입력하여 예측합니다.
| 예측(서빙)은 IRIS Discovery Service의 `serving predict  <http://docs.iris.tools/manual/IRIS-Manual/IRIS-Discovery-Middleware/command/commands/serving.html#serving-predict>`_ 를 사용합니다.

IRIS Discovery Service에서 mnist_test 모델 선택 후, 검색창에 아래 명령어를 입력합니다.

``* | top 50 feature | serving predict mnist_number feature=[(feature, Conv1_input, double, 28, 28,1)] tag=(zero, one, two, three, four, five, six, seven, egiht, nine, ten) version=1``

결과

.. list-table::
   :header-rows: 1

   * - label
     - tag
     - feature
     - predictions
     - probability
     - interpreted
   * - 0,0,0,0,0,1,0,0,0,0
     - five
     - 0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0...
     - [0.62, 0.01, 0.04...]
     - 0.62
     - five
   * - 1,0,0,0,0,0,0,0,0,0
     - zero
     - 0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0...
     - [0.14, 0.03, 0.03...]
     - 0.38
     - zero
   * - ...
     - ...
     - ...
     - ...
     - ...
     - ...
