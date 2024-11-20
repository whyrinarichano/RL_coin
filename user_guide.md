# 사용자 가이드
## 설정 파일
`nntrader/nntrader` 디렉토리 아래에 `net_config.json`이라는 JSON 파일이 있습니다. 이 파일에는 에이전트의 모든 설정이 포함되어 있으며, 프로그램 코드 외부에서 수정할 수 있습니다.

### 네트워크 토폴로지
* `"layers"`
    * CNN의 레이어 목록으로, 출력 레이어도 포함됩니다.
    * `"type"`
        * 가능한 값: {"ConvLayer", "FullyLayer", "DropOut", "MaxPooling", "AveragePooling", "LocalResponseNormalization", "SingleMachineOutput", "LSTMSingleMachine", "RNNSingleMachine"}
    * `"filter shape"`
        * 컨볼루션 레이어의 필터(커널) 모양

* `"input"`
    * `"window_size"`
        * 입력 행렬의 열 개수
    * `"coin_number"`
        * 입력 행렬의 행 개수
    * `"feature_number"`
        * 특징 수 (컴퓨터 비전의 RGB처럼)
        * 가능한 값: {1, 2, 3}
        * 1은 ["close"]를 의미하며, 각 기간의 마지막 가격입니다.
        * 2는 ["close", "volume"]을 의미합니다.
        * 3은 ["close", "high", "low"]를 의미합니다.

### 시장 데이터
* `"input"`
    * `"start_date"`
        * 글로벌 데이터 행렬의 시작 날짜
        * 형식은 yyyy/MM/dd
    * `"end_date"`
        * 글로벌 데이터 행렬의 끝 날짜
        * 형식은 yyyy/MM/dd
        * 성능은 다른 시간 범위에 따라 크게 달라질 수 있습니다.
    * `"volume_average_days"`
        * 코인 선택에 사용될 볼륨의 일 수
    * `"test_portion"`
        * 백테스트 데이터 비율 (0에서 1 사이). 나머지는 학습 데이터입니다.
    * `"global_period"`
        * 거래 기간 및 입력 윈도우 내 가격의 기간.
        * 300(초)의 배수여야 합니다.
    * `"coin_number"`
        * 거래할 자산의 수.
        * 현금(btc)은 포함되지 않습니다.
    * `"online"`
        * 온라인이 아닐 경우, 프로그램은 로컬 데이터베이스에서 코인을 선택하고 입력을 생성합니다.
        * 온라인일 경우, 데이터베이스에 없는 새로운 데이터가 저장됩니다.

## 학습 및 하이퍼파라미터 튜닝
1. 먼저, `nntrader/nntrader/net_config.json` 파일을 수정합니다.
2. 현재 디렉토리가 `nntrader`인지 확인하고, `python main.py --mode=generate --repeat=1`을 입력합니다.
    * 이것은 `train_package` 아래에 1개의 하위 폴더를 생성합니다.
    * 각 하위 폴더에는 `net_config.json`의 사본이 있습니다.
    * `--repeat=n`, n은 양의 정수를 따를 수 있습니다. 각 하위 폴더의 랜덤 시드는 0부터 n-1까지 순차적으로 설정됩니다.
      * 랜덤 시드는 성능에 크게 영향을 미칠 수 있습니다.
3. `python main.py --mode=train --processes=1`을 입력합니다.
    * 방금 생성한 n개의 폴더를 차례로 학습합니다.
    * 데이터를 온라인으로 다운로드하려면 1개 이상의 프로세스를 시작하지 마세요.
    * `"--processes=n"`은 병렬로 n개의 프로세스를 실행함을 의미합니다.
    * tensorflow가 GPU를 지원한다면 `"--device=gpu"`를 추가하세요.
      * GTX1080Ti에서는 4-5개의 학습 프로세스를 동시에 실행할 수 있습니다.
      * GTX1060에서는 2-3개의 학습을 동시에 실행할 수 있습니다.
    * 각 학습 프로세스는 두 단계로 구성됩니다:
      * 사전 학습, 로그 예시:
      
```
INFO:root:average time for data accessing is 0.00070324587822
INFO:root:average time for training is 0.0032548391819
INFO:root:==============================
INFO:root:step 3000
INFO:root:------------------------------
INFO:root:the portfolio value on test set is 2.24213
log_mean is 0.00029086
loss_value is -0.000291
log mean without commission fee is 0.000378

INFO:root:==============================
```
      
      * 롤링 학습을 사용한 백테스트, 로그 예시:
```
DEBUG:root:==============================
INFO:root:the step is 1433
INFO:root:total assets are 17.732482 BTC
```
4. 이후, `nntrader/train_package/train_summary.csv`에서 학습 결과 요약을 확인하세요.
5. 요약을 바탕으로 하이퍼파라미터를 조정하고 1번 단계로 다시 돌아갑니다.

## 로깅
각 학습에는 세 가지 유형의 로그가 있습니다.
* 각 하위 폴더에
    * `programlog`라는 텍스트 파일이 있으며, 실행 중 생성된 로그입니다.
    * `tensorboard` 폴더는 학습 과정에 대한 데이터를 저장하며, tensorboard로 볼 수 있습니다.
        * `tensorboard --logdir=train_package/1`을 입력해 tensorboard를 사용하세요.
* 학습 요약 정보는 네트워크 구성, 검증 세트 및 테스트 세트에서의 포트폴리오 가치 등을 포함하여 `train_package` 폴더 아래의 `train_summary.csv`에 저장됩니다.

## 모델 저장 및 복원
* 학습된 네트워크의 가중치는 `train_package/1`에 `netfile`이라는 이름으로 저장됩니다(파일 3개 포함).

## 데이터 다운로드
* `python main.py --mode=download_data`를 입력하여 학습을 시작하지 않고 데이터를 다운로드할 수 있습니다.
* 프로그램은 `nntrader/nntrader/net_config`의 설정을 사용하여 코인을 선택하고 네트워크 학습에 필요한 데이터를 다운로드합니다.
* 다운로드 속도는 매우 느릴 수 있으며, 중국에서는 오류가 발생할 수 있습니다.
* 데이터를 다운로드할 수 없는 경우, 첫 번째 릴리스에서 제공된 `Data.db` 파일을 데이터베이스 폴더에 넣으세요. `net_config.json`의 `input`에서 `"online"`을 `"false"`로 설정하고 예제를 실행하세요.
  * 이 파일을 사용할 때 입력 데이터 설정(예: `start_date`, `end_date`, `coin_number`)을 변경하지 마세요. 그렇지 않으면 잘못된 결과가 표시될 수 있습니다.

## 백테스트
*참고: 백테스트를 수행하기 전에 알고리즘 학습을 성공적으로 완료해야 합니다.*
* `python main.py --mode=backtest --algo=1`을 입력하여 백테스트를 실행하세요(롤링 학습, 즉 지도 학습에서의 온라인 학습).
* `--algo`는 전통적인 방법의 이름이거나 학습 폴더의 인덱스일 수 있습니다.

## 전통 에이전트
OLPS 요약:

![](https://github.com/DexHunter/nntrader/blob/dev/images/olps_algo.png)

## 플로팅
* 예를 들어, `python main.py --mode=plot --algos=crp,olmar,1 --labels=crp,olmar,nnagent`을 입력해 플로팅하세요.
* `--algos`는 tdagent 알고리즘의 이름이거나 nnagent의 인덱스일 수 있습니다.
* `--labels`는 범례에 표시될 관련 알고리즘의 이름입니다.
* 결과:
![](http://static.zybuluo.com/rooftrellen/u75egf9roy9c2sju48v6uu6o/result.png)

## 백테스트 결과를 테이블로 표시하기
* `python main.py --mode=table --algos=1,olmar,ons --labels=nntrader,olmar,ons`를 입력하세요.
* `--algos`와 `--labels`는 플로팅의 경우와 동일합니다.
* 결과:
```
           average  max drawdown  negative day  negative periods  negative week  portfolio value  positive periods  postive day  postive week  sharpe ratio
nntrader  1.001311      0.225874           781              1378            114        25.022516              1398         1995          2662      0.074854
olmar     1.000752      0.604886          1339              1451           1217         4.392879              1319         1437          1559      0.035867
ons       1.000231      0.217216          1144              1360            731         1.770931              1416         1632          2045      0.032605
```
* `--format` 인수를 사용하여 테이블의 형식을 변경할 수 있습니다. 형식은 `raw`, `html`, `csv`, `latex` 중 하나가 될 수 있으며, 기본값은 `raw`입니다.
---
#가상환경 초기 세팅

##파이썬
conda install python = 3.6
##아나콘다용 텐서플로우 설치안하면 안됨;;
conda install anaconda:tensorflow=1.0.0
conda install contango::tflearn =0.3.2
conda install pympler = 0.5
conda install cvxopt=1.1.9
conda install seaborn==0.8.1
conda install pandas==0.20.3


## packages in environment at /home/cksgh8511/anaconda3/envs/rl_env:
#
# Name                    Version                   Build  Channel
_libgcc_mutex             0.1                        main  
_openmp_mutex             5.1                       1_gnu  
_tflow_select             2.3.0                       mkl  
absl-py                   0.15.0             pyhd3eb1b0_0  
astor                     0.8.1            py36h06a4308_0  
blas                      1.0                    openblas  
c-ares                    1.19.1               h5eee18b_0  
ca-certificates           2024.9.24            h06a4308_0  
certifi                   2021.5.30        py36h06a4308_0  
coverage                  5.5              py36h27cfd23_2  
cvxopt                    1.2.0            py36h2c4e229_0  
cycler                    0.11.0             pyhd3eb1b0_0  
cython                    0.29.24          py36h295c915_0  
dbus                      1.13.18              hb2f20db_0  
expat                     2.6.3                h6a678d5_0  
fftw                      3.3.9                h5eee18b_2  
fontconfig                2.14.1               h52c9d5c_1  
freetype                  2.12.1               h4a9f257_0  
gast                      0.5.3              pyhd3eb1b0_0  
glib                      2.69.1               h4ff587b_1  
glpk                      4.65                 h276157c_3  
gmp                       6.2.1                h295c915_3  
grpcio                    1.36.1           py36h2157cd5_1  
gsl                       2.4                  h14c3975_4  
gst-plugins-base          1.14.1               h6a678d5_1  
gstreamer                 1.14.1               h5eee18b_1  
h5py                      2.10.0           py36hd6299e0_1  
hdf5                      1.10.6               hb1b8bf9_0  
icu                       58.2                 he6710b0_3  
importlib-metadata        4.8.1            py36h06a4308_0  
intel-openmp              2022.1.0          h9e868ea_3769  
jpeg                      9e                   h5eee18b_3  
keras-applications        1.0.8                      py_1  
keras-preprocessing       1.1.2              pyhd3eb1b0_0  
kiwisolver                1.3.1            py36h2531618_0  
lcms2                     2.12                 h3be6417_0  
ld_impl_linux-64          2.40                 h12ee557_0  
lerc                      3.0                  h295c915_0  
libdeflate                1.17                 h5eee18b_1  
libffi                    3.3                  he6710b0_2  
libgcc-ng                 11.2.0               h1234567_1  
libgfortran-ng            7.5.0               ha8ba4b0_17  
libgfortran4              7.5.0               ha8ba4b0_17  
libgomp                   11.2.0               h1234567_1  
libopenblas               0.2.20               h9ac9557_7  
libpng                    1.6.39               h5eee18b_0  
libprotobuf               3.6.0                hdbcaa40_0  
libstdcxx-ng              11.2.0               h1234567_1  
libtiff                   4.5.1                h6a678d5_0  
libuuid                   1.41.5               h5eee18b_0  
libwebp-base              1.3.2                h5eee18b_1  
libxcb                    1.15                 h7f8727e_0  
libxml2                   2.9.14               h74e7548_0  
lz4-c                     1.9.4                h6a678d5_1  
markdown                  3.3.4            py36h06a4308_0  
matplotlib                3.2.2                         0  
matplotlib-base           3.2.2            py36hef1b27d_0  
metis                     5.1.0                hf484d3e_4  
mkl                       2022.1.0           hc2b9512_224  
ncurses                   6.4                  h6a678d5_0  
numpy                     1.15.0           py36h2aefc1b_0  
numpy-base                1.15.0           py36h7cdd4dd_0  
olefile                   0.46                     py36_0  
openjpeg                  2.5.2                he7f1fd0_0  
openssl                   1.1.1w               h7f8727e_0  
pandas                    0.20.3           py36h6022372_2  
patsy                     0.5.1                    py36_0  
pcre                      8.45                 h295c915_0  
pillow                    8.3.1            py36h2c7a002_0  
pip                       21.2.2           py36h06a4308_0  
protobuf                  3.6.0            py36hf484d3e_0  
pympler                   0.5                      py36_0  
pyparsing                 2.4.7              pyhd3eb1b0_0  
pyqt                      5.9.2            py36h05f1152_2  
python                    3.6.13               h12debd9_1  
python-dateutil           2.8.2              pyhd3eb1b0_0  
pytz                      2021.3             pyhd3eb1b0_0  
qt                        5.9.7                h5867ecd_1  
readline                  8.2                  h5eee18b_0  
scipy                     1.1.0           py36_nomklh9c1e066_0  
seaborn                   0.8.1                    py36_0  
setuptools                58.0.4           py36h06a4308_0  
sip                       4.19.8           py36hf484d3e_0  
six                       1.16.0             pyhd3eb1b0_1  
sqlite                    3.45.3               h5eee18b_0  
statsmodels               0.10.1           py36hdd07704_0  
suitesparse               5.2.0                h0253a28_0  
tbb                       2018.0.5             h6bb024c_0  
tensorboard               1.11.0           py36hf484d3e_0  
tensorflow                1.11.0          mkl_py36ha6f0bda_0  
tensorflow-base           1.11.0          mkl_py36h3c3e929_0  
termcolor                 1.1.0            py36h06a4308_1  
tflearn                   0.3.2            py36he51c03d_0    contango
tk                        8.6.14               h39e8969_0  
tornado                   6.1              py36h27cfd23_0  
typing_extensions         4.1.1              pyh06a4308_0  
werkzeug                  1.0.1              pyhd3eb1b0_0  
wheel                     0.37.1             pyhd3eb1b0_0  
xz                        5.4.6                h5eee18b_1  
zipp                      3.6.0              pyhd3eb1b0_0  
zlib                      1.2.13               h5eee18b_1  
zstd                      1.5.6                hc292b87_0 


---
# User Guide
## Configuration File
Under the `nntrader/nntrader` directory, there is a json file called `net_config.json`,
 holding all the configuration of the agent and could be modified outside the program code.
### Network Topology
* `"layers"`
    * layers list of the CNN, including the output layer
    * `"type"`
        * domain is {"ConvLayer", "FullyLayer", "DropOut", "MaxPooling",
        "AveragePooling", "LocalResponseNormalization", "SingleMachineOutput",
        "LSTMSingleMachine", "RNNSingleMachine"}
    * `"filter shape"`
        * shape of the filter (kernal) of the Convolution Layer
* `"input"`
    * `"window_size"`
        * number of columns of the input matrix
    * `"coin_number"`
        * number of rows of the input matrix
    * `"feature_number"`
        * number of features (just like RGB in computer vision)
        * domain is {1, 2, 3}
        * 1 means the feature is ["close"], last price of each period
        * 2 means the feature is ["close", "volume"]
        * 3 means the features are ["close", "high", "low"]

### Market Data
* `"input "`
    * `"start_date"`
        * start date of the global data matrix
        * format is yyyy/MM/dd
    * `"end_date"`
        * start date of the global data matrix
        * format is yyyy/MM/dd
        * The performance could varied a lot in different time ranges.
    * `"volume_average_days"`
        * number of days of volume used to select the coins
    * `"test_portion"`
        * portion of backtest data, ranging from 0 to 1. The left is training data.
    * `"global_period"`
        * trading period and period of prices in input window.
        * should be a multiple of 300 (seconds)
    * `"coin_number"`
        * number of assets to be traded.
        * does not include cash (i.e. btc)
    * `"online"`
        * if it is not online, the program will select coins and generate inputs
        from the local database.
        * if it is online, new data that dose not exist in the database would be saved

## Training and Tuning the hyper-parameters
1. First, modify the `nntrader/nntrader/net_config.json` file.
2. make sure current directory is under `nntrader` and type `python main.py --mode=generate --repeat=1`
    * this will make 1 subfolders under the `train_package`
    * in each subfolder, there is a copy of the `net_config.json`
    * `--repeat=n`, n could followed by any positive integers. The random seed of each the subfolder is from 0 to n-1 sequentially.
      * Notably, random seed could also affect the performance in a large scale.
3. type `python main.py --mode=train --processes=1`
    * this will start training one by one of the n folder created just now
    * do not start more than 1 processes if you want to download data online
    * "--processes=n" means start n processes running parallely.
    * add "--device=gpu" if your tensorflow support gpu.
      * On GTX1080Ti you should be able to run 4-5 training process together.
      * On GTX1060 you should be able to run 2-3 training together.
    * Each training process is made up from 2 stages:
      * Pre-training, log example:
      
      
```
INFO:root:average time for data accessing is 0.00070324587822
INFO:root:average time for training is 0.0032548391819
INFO:root:==============================
INFO:root:step 3000
INFO:root:------------------------------
INFO:root:the portfolio value on test set is 2.24213
log_mean is 0.00029086
loss_value is -0.000291
log mean without commission fee is 0.000378

INFO:root:==============================

```
        
        
      * Backtest with rolling train, log example:
```
        DEBUG:root:==============================
INFO:root:the step is 1433
INFO:root:total assets are 17.732482 BTC
```
4. after that, check the result summary of the training in `nntrader/train_package/train_summary.csv`
5. tune the hyper-parameters based on the summary, and go to 1 again.

## Logging
There are three types of logging of each training.
* In each subfolder
    * There is a text file called `programlog`, which is the log generated by the running programming.
    * There is a `tensorboard` folder saves the data about the training process which could be viewed by tensorboard.
        * type `tensorboard --logdir=train_package/1` to use tensorboard
* The summary infomation of this training, including network configuration, portfolio value on validation set and test set etc., will be saved in the `train_summary.csv` under `train_pakage` folder

## Save and Restore of the Model
* The trained weights of the network are saved at `train_package/1` named as `netfile` (including 3 files). 

## Download Data
* Type `python main.py --mode=download_data` you can download data without starting training
* The program will use the configurations in `nntrader/nntrader/net_config` to select coins and
  download necessary data to train the network.
* The downloading speed could be very slow and sometimes even have error in China.
* For those who cann't download data, please check the first release where there is a `Data.db` file, put it in the database folder. Make sure the `online` in `input` in `net_config.json` to be `false` and run the example.
  * Note that using the this file, you shouldn't make any changes to input data configuration(For example `start_date`, `end_date` or `coin_number`) otherwise incorrect result might be presented.
  
## Back-test
*Note: Before back-testing, you need to suceessfully finish training of algo first*
* Type `python main.py --mode=backtest --algo=1` to execute
backtest with rolling train(i.e. online learning in supervised learning)
on the target model.
* `--algo` could be either the name of traditional method or the index of training folder

## Tradition Agent
OLPS summary:

![](https://github.com/DexHunter/nntrader/blob/dev/images/olps_algo.png)

## Plotting
* type `python main.py --mode=plot --algos=crp,olmar,1 --labels=crp,olmar,nnagent
`,for example, to plot
* `--algos` could be the name of the tdagent algorithms or
the index of nnagent
* `--labels` is the name of related algorithm that will be shown in the legend
* result is
![](http://static.zybuluo.com/rooftrellen/u75egf9roy9c2sju48v6uu6o/result.png)

## present backtest results in a table
* type `python main.py --mode=table --algos=1,olmar,ons --labels=nntrader,olmar,ons`
* `--algos` and `--lables` are the same as in plotting case
* result:
```
           average  max drawdown  negative day  negative periods  negative week  portfolio value  positive periods  postive day  postive week  sharpe ratio
nntrader  1.001311      0.225874           781              1378            114        25.022516              1398         1995          2662      0.074854
olmar     1.000752      0.604886          1339              1451           1217         4.392879              1319         1437          1559      0.035867
ons       1.000231      0.217216          1144              1360            731         1.770931              1416         1632          2045      0.032605

```
* use `--format` arguments to change the format of the table,
 could be `raw` `html` `csv` or `latex`. The default one is raw.
