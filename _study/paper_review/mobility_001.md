---
layout: posts
title: "A Reinforcement Learning Framework for PQoS in a Teleoperated Driving Scenario"
subject: mobility
category: paper_review
description: "선행연구 - Artificial Intelligence in Vehicular Wireless Networks: A Case Study Using ns-3"
---
# RAN-AI

<a href ="https://github.com/signetlabdei/ns3-ran-ai" target="_blank" rel="noopener noreferrer"><img src="https://img.icons8.com/ios-glyphs/120/null/github.png" width="15" height="15" style="box-shadow:none;"></a> [**ns3-ran-ai**](https://github.com/signetlabdei/ns3-ran-ai){:target="_blank"}

Matteo Drago 외 연구진들은 RAN-AI라는 ns3환경에서 시뮬레이션 되는 새로운 엔티티를 개발하였다.

매우 동적인 V2X 시스템에서 QoS는 언제든 예기치 못하게 변경 또는 저하될 수 있다. 사람의 안전과 직결되는 교통수단에서 통신 장애는 치명적인 결과를 초래할 수 있다. 이를 위해 QoS를 사전에 예측하여 그에 따라 애플리케이션을 통제한다는 개념인 PQoS(Predictive Quality of Service)를 도입했다. 바로 RAN-AI는 V2X 네트워크를 위한 PQoS 메커니즘이다.

## 전체적인 RAN-AI의 동작 과정

1. 애플리케이션 통계를 가져오고, gNB와 최종 사용자로부터 RL(강화학습) 에이전트의 [입력]({{ page.url }}#pqos-입력)으로 사용가능한 RAN 측정값(full-stack)을 수집한다.
2. RL 에이전트는 측정값(입력)을 바탕으로 전체 성능을 최대화 하는 최적의 [대응]({{ page.url }}#pqos-대응)을 결정한다.
3. 최종 사용자에게 QoS 변동사항을 알리고, 에이전트의 결정을 바탕으로 대응책을 제시한다.

## PQoS **입력**

- Context 정보:<br>
    운영 시나리오, 도로의 요소, 네트워크 구축, 시간에 대한 통합 정보
- 사용자 궤적:<br>
    드라이빙 애플리케이션은 최종 사용자의 궤적을 네트워크에 제공할 수 있으며, RAN-AI는 사용자가 위치하게 될 gNB에 대한 데이터(e.g., cell load)를 습득할 수 있다.
- 교통 정보:<br>
    RAN-AI는 외부 제어 센터로부터 트래픽 상태를 수집하고 데이터를 바탕으로 트래픽 예측을 제공할 수 있다.
- 네트워크 메트릭:<br>
    RAN-AI는 PHY 및 MAC 계층(e.g., RSRP, RSRQ, SINR, PRBs utilization, MCS), RLC 및 PDCP 계층(e.g., 사용자간 데이터 트래픽 통계)에서 측정된 데이터들을 얻을 수 있다.
- 상위 계층 메트릭:<br>
    RAN-AI는 사용자의 애플리케이션을 통해 E2E(end-to-end) 성능(e.g., delay와 throughput의 평균, 표준편차, min, max)에 대해 알 수 있다.

## PQoS **대응**

RAN-AI는 PQoS 입력을 바탕으로 적절한 조치(e.g., 스케쥴링 결정 변경, propagation condition의 function에 따른 무선자원할당 적용, 네트워크 용량에 기반한 traffic request 변조, system numerology 수정을 통한 탄력적인 통신 채널 제공)를 취해야 한다.

또 다른 방법으로는 전송 전 애플리케이션 계층에서 생성되는 데이터의 크기를 줄이는 것이다.
<br><br>

# 시뮬레이션

RAN-AI 시뮬레이션 수행은 ns-3를 통해 이루어졌다. 이는 ns3-gym 프레임워크를 확장한 형태이다.

## 구조

<img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/1.png" alt="<b>[Fig. 1]</b> O-RAN 호환 architecture 및 workflow <a href='#Reference'>[1]</a>."/>

- [SUMO(Simulation of Urban MObility)](https://eclipse.dev/sumo/){:target="_blank"}<br>
    시뮬레이션 차량들은 SUMO를 통해 생성된 실제 길을 따라 이동.
- [GEMV 2](https://vehicle2x.net/){:target="_blank"}<br>
    V2V, V2I 통시을 위한 지정학적 propagation model.<br>

## *Setup*

논문 참고



## SUMO

먼저 OpenStreetMap과 같은 오픈소스 지도에서 osm 파일을 가져온다.

<img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/2.png" alt="<b>[Fig. 2]</b> OpenStreetMap으로 지도 파일 내보내기."/>

‘내보내기’ 버튼을 눌러 직접 파일을 저장할 수도 있고,

```bash
wget -O inputPolygon/SanFrancisco.osm “https://api.openstreetmap.org/api/0.6/map?bbox=-122.4115,37.7814,-122.3899,37.7965"
```

으로 CLI를 통해 받아오는 것도 가능하다. 그런데 이때 파일을 'inputPolygon' 디렉토리에 저장하는 것은 GEMV [다운로드](https://vehicle2x.net/download/){:target="_blank"}를 완료한 상태에서 하는건데, 다른 디렉토리에서 먼저 아래 과정을 수행하고 파일들을 나중에 'inputPolygon' 으로 복사해도 상관은 없다.

이렇게 해서 저장된 `.osm` 파일을 xml 파일로 변환해주어야 한다. SUMO에는 여러 종류의 xml 파일이 있는데, 교통 인프라 네트워크인 `net.xml`, 루트 파일인 `trips.xml` 및 `rou.xml` 등이 있다. 우선 netconfverter를 이용하여 osm 파일을 `net.xml` 파일로 변환해 주어야 한다.

```bash
netconvert --osm inputPolygon/ SanFrancisco.osm -o inputPolygon/SanFrancisco.net.xml --geometry.remove --ramps.guess --junctions.join --tls.guess-signals --tls.discard-simple --tls.join --remove-edges.by-type railway.subway
```

<img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/3.png" alt="<b>[Fig. 3]</b> SUMO-gui를 통해 확인한 SanFrancisco.net.xml."/>

그 후 `randomTrips.py`를 이용해 랜덤하게 차량들의 경로를 생성하고, `trips.xml` 로 저장한다. 이 때 차량의 개수를 저장해준다.

```bash
randomTrips.py -n inputPolygon/ SanFrancisco.net.xml -e 20 -o inputPolygon/SanFrancisco.trips.xml
```

그 후 duarouter를 이용해 `rou.xml` 파일을 생성한다.

```bash
duarouter -n inputPolygon/SanFrancisco.net.xml --route-files inputPolygon/SanFrancisco.trips.xml -o inputPolygon/SanFrancisco.rou.xml --ignore-errors
```

시뮬레이션에 필요한 파일 경로와 같은 기본 정보나 시뮬레이션 시간 등의 파라미터는 `sumo.cfg` 파일에 저장해 준 후, 이 `sumo.cfg` 파일을 이용해 SUMO simulation을 실행한다.

```bash
sumo -c SanFrancisco.sumo.cfg
```

위 `SanFrancisco.sumo.cfg` 파일에서 알 수 있듯 시뮬레이션 결과는 inputMobilitySUMO 폴더 안에 `SanFrancisco-mobility-trace.xml` 파일로 저장된다.

그럼 이제 matlab으로 GEMV 시뮬레이션을 실행할 준비가 된것이다.

OpenStreetMap을 이용해 바로 파일을 받아오는 방법 외에, `osmWebWizard.py`를 사용하는 방법이 있다. `osmWebWizard.py`를 통해 더 간편하게 지도를 다운로드 받을 수 있다. 파일의 위치를 찾기 힘들다면 `find`를 이용하자.

```bash
sudo find / -n osmWebWizard.py
```

<img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/3-2.png" alt=""/>

## GEMV

만들어 둔 xml 파일을 돌리기 위해 우선 `simSetting.m` 파일에 SanFracisco case를 추가해주고, 시뮬레이션을 실행한다.

```matlab
>> runSimulation
```

시뮬레이션이 완료되면 kml 파일과 csv 파일이 'outputKML', 'outputSim' 디렉토리에 저장된다.

Kml 파일은 구글어스로 시각화 할 수 있다.


<div class="post__stage-container">
    <div class="post__stage">
        <img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/4.png" alt="<b>[Fig. 4]</b> Busan."/>
    </div>
    <div class="post__stage">
        <img class="modal img__medium" src="/_pages/study/paper_review/images/mobility_001/5.png" alt="<b>[Fig. 5]</b> San Francisco."/>
    </div>
</div>



## RAN-AI

Ran-ai 환경을 구축하기 위해 Anaconda를 이용하는 것이 좋다.

```bash
sudo apt-get install -y libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6
# https://docs.anaconda.com/free/anaconda/allpkglists/
curl --output anaconda.sh https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
shasum -a 256 anaconda.sh # 무결성 확인
bash ./anaconda.sh
```

```bash
sudo nano ~/.bashrc
```

마지막 줄에 `export PATH=~/anaconda3/bin:~/anaconda3/condabin:$PATH` 추가해준다.

```bash
source ~/.bashrc
```

Ran-ai 코드를 받아준다.

```bash
git clone https://github.com/signetlabdei/ns3-ran-ai.git
```

그 후 conda 환경 세팅을 해준다.

```bash
conda create -n ns3 python=3.10
conda activate ns3
```

```bash
conda install numpy tensorflow pytorch pandas seaborn matplotlib=3.6 psutil ipywidgets conda-forge::tikzplotlib
pip install pickle5 sem
```

Python interface를 추가해준다.

```bash
cd contrib/ns3-ai/py_interface
pip3 install . --user
cd ../../..
```

Python interface를 위한 환경 변수를 추가해준다.

```bash
export PYTHONPATH=/home/host/.local/lib/python3.10/site-packages/ns3_ai-0.0.2-py3.10-linux-x86_64.egg:$ PYTHONPATH
```

- **Offline learning**

    ```bash
    python3 scratch/ran-ai/get_offline_stats.py -run
    python3 scratch/ran-ai/run.py -train -run -offline -episode 100
    ```

    오프라인 데이터를 생성하기 위해 `get_offline_stats.py` 파일을 실행한다.

    `get_offline_stats.py`를 run 할 때, 여러가지 error 상황에 부딪히게 된다. 우선 python 코드 내부의 경로를 수정해주어야 한다. Import 구문 사이에 `sys.path.append(os.path.abspath('.'))`를 추가해 준다. `params_grid` 리스트에는 `gemvTracesPath`와 `appTracesPath`로 파일 경로를 명시해 주고 있다. 이를 내가 사용하는 실제 파일 경로로 변경해 주었다.

    그 후 ns3 실행 파일인 `waf`가 있는 폴더를 설정해주어야 한다. `ns_path = '/home/host/ns3-ran-ai'`로 설정해준 후 코드를 실행시키게 되면 error: ‘numeric_limits’ is not a member of ‘std’라는 std 에러가 발생한다. 이 에러는 `csv-reader.cc`파일의 오류로 `csv-reader.h`파일에 `<stddef.h>` 파일과 `<limits>` 파일을 include함으로써 해결 가능하다. &rarr; [ns3 설치](/study/communication/ns3/2/) 참고.

- **Online learning**

    ```bash
    python3 scratch/ran-ai/run.py -train -run -episode 100
    ```

    Simulation.py의 `initialize_simulation()` 함수에 의해 “TRAINING”이라는 단어가 출력되고 os.makedirs() 함수에 의해 `/output/train/` 및 하위 폴더가 새롭게 생성된다.

    처음 training을 실행시키기 위해 parser에 `-run`을 활성해 해주었고, `get_input()`함수의 `running = args['run']`에 의해 `running`이라는 변수가 True가 된다.

    `initialize_simulation`이 완료되면, 메모리 설정을 위해 `Experiment()`함수가 실행된다. 이 함수로 생성된 설정값들은 `experiment`라는 변수에 저장되게 된다. 이 때 Experiment()함수 4번째 파라미터를 `ns3-ran-ai`의 경로로 설정해준다.

    `experiment`설정이 완료되면 `initialize_online_episode()`함수가 실행된다.






---
# <a name="Reference"></a>Reference

1. M. Drago, F. Mason, T. Zugno, M. Giordani, M. Boban and M. Zorzi, "Artificial Intelligence in Vehicular Wireless Networks: A Case Study Using ns-3", Workshop on ns-3 (WNS3), 2022.
2. M. Boban, J. Barros and O. K. Tonguz, "Geometry-Based Vehicle-to-Vehicle Channel Modeling for Large-Scale Simulation," in IEEE Transactions on Vehicular Technology, vol. 63, no. 9, pp. 4146-4164, Nov. 2014, doi: 10.1109/TVT.2014.2317803.
{:.post__reference}


