"""이 구현은 우리의 논문, 금융 포트폴리오 관리 문제를 위한 심층 강화 학습 프레임워크([arXiv:1706.10059](https://arxiv.org/abs/1706.10059))의 구현이며, 포트폴리오 관리 연구를 위한 도구 모음입니다.

* 논문에서 설명한 정책 최적화 방법은 포트폴리오 관리 문제에 특화되어 설계되었습니다.
  * 일반적인 강화 학습 알고리즘과 달리, 이 방법은 지도 학습과 유사한 효율성과 안정성을 가지고 있습니다.
  * 이는 문제를 거래 비용으로 정규화된 즉시 보상 최적화 문제로 공식화했기 때문이며, 몬테카를로 방식이나 부트스트랩 방식으로 경사를 추정할 필요가 없습니다.
* 사용자는 별도의 JSON 파일에서 네트워크 구조, 학습 방법 또는 입력 데이터를 구성할 수 있습니다. 학습 과정은 기록되며, 사용자는 tensorboard를 통해 학습 과정을 시각화할 수 있습니다.
결과 요약 및 병렬 학습을 통해 하이퍼파라미터 최적화를 더 잘 수행할 수 있습니다.
* 또한 이 라이브러리에는 재무 모델 기반 포트폴리오 관리 알고리즘이 포함되어 있어 비교 연구가 가능합니다. 구현은 Li와 Hoi의 툴킷 [OLPS](https://github.com/OLPS/OLPS)를 기반으로 합니다.

## 논문 버전과의 차이점
이 라이브러리는 우리의 주요 프로젝트의 일부이며, 논문에서 공개한 버전보다 여러 차례 업데이트된 상태입니다.

* 이번 버전에서는 몇 가지 기술적 버그가 수정되었고, 하이퍼파라미터 튜닝 및 엔지니어링에서 개선이 이루어졌습니다.
  * arXiv v2 논문에서 가장 중요한 버그는 언급된 테스트 기간이 실제 실험보다 약 30% 짧았다는 점입니다. 이로 인해 볼륨-관찰 간격(자산 선택을 위한)이 논문에서 백테스트 데이터와 중복되었습니다.
* 새로운 하이퍼파라미터를 통해 사용자는 짧은 시간 내에(30분 이내) 모델을 학습할 수 있습니다.
* 모든 업데이트는 향후 논문 버전에 통합될 예정입니다.
* 오픈 소스 버전에서는 원래의 버전 기록과 내부 논의, 일부 코드 내 주석이 제거되었습니다. 여기에는 구현되지 않은 아이디어들이 포함되어 있으며, 일부는 향후 출판물의 기초가 될 가능성이 높습니다.

## 플랫폼 지원
Windows에서는 Python 3.5+, Linux에서는 Python 2.7+/3.5+ 버전이 지원됩니다.

## 종속성
`pip install -r requirements.txt` 명령을 통해 종속성을 설치할 수 있습니다.

* tensorflow (>= 1.0.0)
* tflearn
* pandas
* ...

## 사용자 가이드
[사용자 가이드](user_guide.md)를 참고해주세요.

## 감사의 말
이 프로젝트는 다음 오픈 소스 프로젝트의 코드를 사용하지 않았다면 완성될 수 없었습니다:
* [Online Portfolio Selection toolbox](https://github.com/OLPS/OLPS)

## 커뮤니티 기여
커뮤니티의 기여를 환영합니다. 기여에는 다음이 포함되지만 이에 국한되지 않습니다:
* 버그 수정
* 주식, 선물, 옵션과 같은 다른 시장에 대한 인터페이스 추가
* 중개 API 추가 (`marketdata` 폴더 아래)
* 더 많은 백테스트 전략 추가 (`tdagent` 폴더 아래)

## 위험 고지 (실거래 시)
거래에는 항상 손실의 위험이 있습니다. **모든 거래 전략은 사용자 자신의 책임 하에 사용됩니다.**

* 이 프로젝트는 2017년부터 오픈 소스로 공개되었습니다. 시장 효율성이 그 이후로 상당히 높아졌을 수 있습니다. 동일한 알고리즘이 여전히 작동한다고 보장할 수 없습니다.
* 실제 시장 상황에 가깝게 가정하려고 노력했지만, 모든 결과는 정적 역사 데이터에 대한 백테스트에서 얻은 것입니다. 이를 실거래 봇으로 배포할 경우 미끄러짐(slippage)이나 시장 영향이 문제가 될 수 있습니다.
----
This is the implementation of our paper, A Deep Reinforcement Learning Framework for the Financial Portfolio Management Problem ([arXiv:1706.10059](https://arxiv.org/abs/1706.10059)), together with a toolkit of portfolio management research.

* The policy optimization method we described in the paper is designed specifically for portfolio management problem.
  * Differing from the general-purpose reinforcement learning algorithms, it has similar efficiency and robustness to supervized learning.
  * This is because we formulate the problem as an immediate reward optimization problem regularised by transaction cost, which does not require a monte-carlo or bootstrapped estimation of the gradients.
* One can configurate the topology, training method or input data in a separate json file. The training process will be recorded and user can visualize the training using tensorboard.
Result summary and parallel training are allowed for better hyper-parameters optimization.
* The financial-model-based portfolio management algorithms are also embedded in this library for comparision purpose, whose implementation is based on Li and Hoi's toolkit [OLPS](https://github.com/OLPS/OLPS).

## Differences from the article version
Note that this library is a part of our main project, and it is several versions ahead of the article.

* In this version, some technical bugs are fixed and improvements in hyper-parameter tuning and engineering are made.
  * The most important bug in the arxiv v2 article is that the test time-span mentioned is about 30% shorter than the actual experiment. Thus the volumn-observation interval (for asset selection) overlapped with the backtest data in the paper.
* With new hyper-parameters, users can train the models with smaller time durations.(less than 30 mins)
* All updates will be incorporated into future versions of the paper.
* Original versioning history,  and internal discussions, including some in-code comments, are removed in this open-sourced edition. These contains our unimplemented ideas, some of which will very likely become the foundations of our future publications

## Platform Support
Python 3.5+ in windows and Python 2.7+/3.5+ in linux are supported.

## Dependencies
Install Dependencies via `pip install -r requirements.txt`

* tensorflow (>= 1.0.0)
* tflearn
* pandas
* ...

## User Guide
Please check out [User Guide](user_guide.md)

## Acknowledgement
This project would not have been finished without using the codes from the following open source projects:
* [Online Portfolio Selection toolbox](https://github.com/OLPS/OLPS)

## Community Contribution
We welcome contributions from the community, including but not limited to:
* Bug fixing
* Interfacing to other markets such as stock, futures, options
* Adding broker API (under `marketdata`)
* More backtest strategies (under `tdagent`)

## Risk Disclaimer (for Live-trading)

There is always risk of loss in trading. **All trading strategies are used at your own risk**

* The project has been open-sourced for many years (since 2017). The market efficiency may have increased quite a lot since then. There is no guarantee the exact same algorithm can still work.
* Although we tried to make assumptions closer to the situation in the real market, the results are all from backtest on static historical data. The slippage or market impact can always be a problem if you deploy it as a live trading bot.
