# [序言]
## 安装tensorflow前系统环境准备：
  ### 安装系统依赖：
  ```shell

  sudo apt-get install cmake
  sudo apt-get install git

  sudo apt update
  sudo apt install libatlas-base-dev  # 此命令必选，注意！！！
  ```

  #### 安装虚拟环境(已经安装了虚拟环境可跳过)：
  ```shell
    sudo apt install python3-dev python3-pip  # 如已经安装了Python3和pip3则跳过此命令
    sudo pip3 install -U virtualenv  # 如果已经安装了虚拟环境，跳过此命令
  ```

  #### 安装python3依赖包
  ```shell
    pip install keras_applications==1.0.8 --no-deps
    pip install keras_preprocessing==1.1.0 --no-deps
    pip install h5py==2.9.0
    pip install -U six wheel mock
  ```


  #### 安装过程中出现scipy的模块错误，应该是没有 Fortran 编译器
  ```shell
    sudo apt update
    sudo apt install gfortran -y
  ```

# [安装]
## 1. 安装后遇到tensorflow问题(报错)

#  一、 tensorflow==2.11.0报错信息与解决办法:
### 一、 没有找到[libtensorflow_io.so]模块错误:  
  - 在安装 tensorflow 后 python3中测试报错信息中为 The file to load file system plugin from does not exist.:[libtensorflow_io.so] 即为 [libtensorflow_io.so] 模块没有找到,通过以下步骤解决
  - 在 python 中测试结果如下:

  ```python
  >>> import tensorflow as tf
  /home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/__init__.py:98: UserWarning: unable to load libtensorflow_io_plugins.so: unable to open file: libtensorflow_io_plugins.so, from paths: ['/home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/libtensorflow_io_plugins.so']
  caused by: ["[Errno 2] The file to load file system plugin from does not exist.: '/home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/libtensorflow_io_plugins.so'"]
    warnings.warn(f"unable to load libtensorflow_io_plugins.so: {e}")
  /home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/__init__.py:104: UserWarning: file system plugins are not loaded: unable to open file: libtensorflow_io.so, from paths: ['/home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/libtensorflow_io.so']
  caused by: ['/home/administrator/Desktop/aiStudy/lib/python3.7/site-packages/tensorflow_io/python/ops/libtensorflow_io.so: cannot open shared object file: No such file or directory']
    warnings.warn(f"file system plugins are not loaded: {e}")
  ```

#### 2. 测试结果总结:
  ```shell
  1. 无法加载libtensorflow_io_plugins.so
  2. 无法加载libtensorflow_io.so
  3. 怀疑是tensorflow-io-gcs-filesystem 包的问题
  ```
              
### 二、 解决办法:
1. 安装 pipdeptree 工具查看包关系(pip install pipdeptree)
2. 通过 pipdeptree 工具中的 pipdeptree | grep [包名] 指令来查看关系,如 pipdeptree | grep tensorflow
3. 返回结果如下:
```shell
 (RPI_SDK) zzy@raspberrypi:~/RPI_SDK/use_material_lib $ pipdeptree | grep tensorflow
 tensorflow==2.11.0
 tensorflow-cpu-aws [required: ==2.11.0, installed: 2.11.0]
   - tensorflow-estimator [required: >=2.11.0,<2.12, installed: 2.11.0]
   - tensorflow-io-gcs-filesystem [required: >=0.23.1, installed: 0.29.0]
```
#### 4. 简化关系图如下:
```shell
  tensorflow==2.11.0
    ├── tensorflow-cpu-aws==2.11.0
    │   └── (特定于AWS CPU的TensorFlow实现)
    ├── tensorflow-estimator==2.11.0
    │   └── (用于模型训练和评估的高级API)
    └── tensorflow-io-gcs-filesystem==0.29.0
        └── (用于与Google Cloud Storage交互的TensorFlow I/O扩展)
```
         
#### 5. 正确的关系图如下:
```shell
   - (RPI_SDK) zzy@raspberrypi:~/RPI_SDK/use_material_lib $ pipdeptree | grep tensorflow
     Warning!!! Possibly conflicting dependencies found:
     * tensorflow-cpu-aws==2.11.0
     - tensorflow-io-gcs-filesystem [required: >=0.23.1, installed: ?]
     ------------------------------------------------------------------------
     tensorflow-cpu-aws==2.11.0
       - tensorflow-estimator [required: >=2.11.0,<2.12, installed: 2.11.0]
       - tensorflow-io-gcs-filesystem [required: >=0.23.1, installed: ?]
```

#### 6. 简化后的包关系图如下:
  ```shell
    tensorflow==2.11.0
    ├── tensorflow-cpu-aws==2.11.0
    │   └── tensorflow-estimator==2.11.0
    │       └── tensorflow-io-gcs-filesystem==0.29.0
    └── tensorflow-estimator==2.11.0
  ```

#### 7. 测试结果总结：
  
   - 发现tensorflow-io-gcs-filesystem 包存在问题
   - 包的依赖关系为 TensorFlow 2.11.0 依赖于 tensorflow-cpu-aws 2.11.0，后者又依赖于 tensorflow-estimator 2.11.0 和 tensorflow-io-gcs-filesystem 0.29.0
   - 卸载 tensorflow-io-gcs-filesystem 包
   


#### 8. 卸载 tensorflow-io-gcs-filesystem 包
  - 通过命令 pip uninstall tensorflow-io-gcs-filesystem 卸载
  - 卸载后再次测试输入 pipdeptree | grep tensorflow
  - 返回结果如下:
    ```shell
      (RPI_SDK) zzy@raspberrypi:~/RPI_SDK/use_material_lib $ pipdeptree | grep tensorflow
      Warning!!! Possibly conflicting dependencies found:
      * tensorflow-cpu-aws==2.11.0
      - tensorflow-io-gcs-filesystem [required: >=0.23.1, installed: ?]
      ------------------------------------------------------------------------
      tensorflow-cpu-aws==2.11.0
        - tensorflow-estimator [required: >=2.11.0,<2.12, installed: 2.11.0]
        - tensorflow-io-gcs-filesystem [required: >=0.23.1, installed: ?]
    ```


  - 在python3中测试，输入以下代码:
    ```python
    import tensorflow as tf # 导入TensorFlow库
    print(tf.__version__) # 打印TensorFlow的版本
    ```
  - 返回结果如下: 
    ```python
      - >>> import tensorflow as tf
      - >>> print(tf.__version__)
      - 2.11.0
    ```


  - 测试结果总结:
    - 卸载 tensorflow-io-gcs-filesystem 包后，tensorflow 2.11.0 依赖关系正常
    - 在删除 tensorflow-io-gcs-filesystem 包后，上层依赖关系正常，再无因为 tensorflow-io-gcs-filesystem 包而导致的错误
                

# 二、 tensorflow==1.14.0报错报错信息与解决办法：
### 1. 导入 [protobuf] 模块错误:
####  1. 报错信息如下(protobuf 模块报错)<br>

  在python3中测试，输入以下代码:
  ```python
    import tensorflow as tf
  ```
####  返回信息如下：
  ```shell
  >>> import tensorflow as tf
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/__init__.py", line 28, in <module>
      from tensorflow.python import pywrap_tensorflow  # pylint: disable=unused-import
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/__init__.py", line 52, in <module>
      from tensorflow.core.framework.graph_pb2 import *
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/core/framework/graph_pb2.py", line 16, in <module>
      from tensorflow.core.framework import node_def_pb2 as tensorflow_dot_core_dot_framework_dot_node__def__pb2
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/core/framework/node_def_pb2.py", line 16, in <module>
      from tensorflow.core.framework import attr_value_pb2 as tensorflow_dot_core_dot_framework_dot_attr__value__pb2
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/core/framework/attr_value_pb2.py", line 16, in <module>
      from tensorflow.core.framework import tensor_pb2 as tensorflow_dot_core_dot_framework_dot_tensor__pb2
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/core/framework/tensor_pb2.py", line 16, in <module>
      from tensorflow.core.framework import resource_handle_pb2 as tensorflow_dot_core_dot_framework_dot_resource__handle__pb2
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/core/framework/resource_handle_pb2.py", line 42, in <module>
      serialized_options=None, file=DESCRIPTOR),
    File "/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/google/protobuf/descriptor.py", line 561, in __new__
      _message.Message._CheckCalledFromGeneratedFile()
  TypeError: Descriptors cannot not be created directly.
  If this call came from a _pb2.py file, your generated code is out of date and must be regenerated with protoc >= 3.19.0.
  If you cannot immediately regenerate your protos, some other possible workarounds are:
  1. Downgrade the protobuf package to 3.20.x or lower.
  2. Set PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python (but this will use pure-Python parsing and will be much slower).
  ```

#### 2. 测试结果总结:
  ```shell
  1. 报错信息为 tensorflow==1.14.0 版本不支持 protobuf>=3.20.0
  2. 解决办法为降级 protobuf 版本，或通过设置 PROTOCOL_BUPFERS_PYTHON_iMPLEMENTATION=PYTHON
  ```

#### 1. [protobuf] 模块降低版本的办法：
  ```shell
  - 降级 protobuf 版本命令如下:
  - pip install protobuf==3.19.0 或 pip install 'protobuf~=3.19.0'
  ```
            
#### 1. 降级后 [protobuf] 版本测试结果总结:
 ```python
   Python 3.7.16 (default, Jan 17 2023, 15:51:26) 
   [GCC 11.2.0] :: Anaconda, Inc. on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>> import tensorflow as tf
   >>> print(tf.__version__)
   1.14.0
 ```

#### 1. 测试结果总结： 
 - 降级 protobuf 版本后，tensorflow==1.14.0 依赖关系正常
 - 降级 protobuf 版本后，上层依赖关系正常，再无因为 tensorflow==1.14.0 包而导致的错误



<!--           
Python 3.7.16 (default, Jan 17 2023, 15:51:26) 
[GCC 11.2.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:516: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint8 = np.dtype([("qint8", np.int8, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:517: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_quint8 = np.dtype([("quint8", np.uint8, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:518: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint16 = np.dtype([("qint16", np.int16, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:519: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_quint16 = np.dtype([("quint16", np.uint16, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:520: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint32 = np.dtype([("qint32", np.int32, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorflow/python/framework/dtypes.py:525: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  np_resource = np.dtype([("resource", np.ubyte, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:541: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint8 = np.dtype([("qint8", np.int8, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:542: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_quint8 = np.dtype([("quint8", np.uint8, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:543: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint16 = np.dtype([("qint16", np.int16, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:544: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_quint16 = np.dtype([("quint16", np.uint16, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:545: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  _np_qint32 = np.dtype([("qint32", np.int32, 1)])
/home/zzy/.conda/envs/RPI_SDK_CS/lib/python3.7/site-packages/tensorboard/compat/tensorflow_stub/dtypes.py:550: FutureWarning: Passing (type, 1) or '1type' as a synonym of type is deprecated; in a future version of numpy, it will be understood as (type, (1,)) / '(1,)type'.
  np_resource = np.dtype([("resource", np.ubyte, 1)]) -->
