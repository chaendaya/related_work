

## 액터 모델 (Actor Model) 
1973년  Carl Hewitt가 Peter Bishop, Richard Steiger와 함께 최초로 제안한 계산 모델이다. <br>
액터(actor)를 컴퓨팅의 기본 단위로 삼아 복수의 연산이 병렬로 안전하게 수행될 수 있도록 하는 이론적 프레임워크이다. <br>
이후 Gul Agha 등 여러 연구자에 의해 이론적, 실용적으로 확장·정교화되었다.

**등장 배경** <br>
1970년대 초반은 인공지능(AI) 분야뿐 아니라 컴퓨터 과학 전반적으로 병렬성과 동시성 지원이 초기 단계에 머물던 시기였다. 
이미 다중 프로세서·병렬 컴퓨터 아키텍처(예: ILLIAC IV 등) 실험이 있었으나, 
당시 AI 연구 커뮤니티의 주류 프로그램 모델은 대부분 순차적(Sequential)으로 설계되고 실행되었다.
기존의 순차적 프로그램 모델로는 수많은 독립적 프로세서(혹은 프로세스, 스레드)가 동시에 협력해야 하는 상황에 적합하지 않았다.
병렬성, 동시성, 분산성, 그리고 오류격리(fault isolation) 문제 등을 명확하게 해결할 추상화 구조가 필요했다.

- **A Universal Modular ACTOR Formalism for Artificial Intelligence** [PDF](https://www.ijcai.org/Proceedings/73/Papers/027B.pdf) <br>
Carl Hewitt, Peter Bishop, Richard Steiger, IJCAI 1973 <br>
AI 시스템을 독립적이고 메시지 기반으로 작동하는 작은 단위(액터)들로 짜며, 그 내부 동작 규칙을 일관되게 정의하여 
구조적으로 깔끔한(명령식·저수준 제어가 없는) 병렬 인공지능 시스템을 설계하는 방법론을 제안

<br>

Gul Agha는 1980년대에 액터 모델을 더욱 확장하고 형식화하였다.
특히, Agha는 액터 모델을 동시성 객체지향 프로그래밍의 기반으로 재정립하며 세 가지 주요 연산(create, send, become)으로 모델을 간결하고 강력하게 정리했습니다. 
그의 박사 논문(1985년)과 1986년 MIT Press에서 출간된 저서가 표준 참조로 널리 인용된다.

- **Actors: A Model of Concurrent Computation in Distributed Systems** [한국어 번역](https://github.com/chaendaya/related_work/blob/9e9e8c95465aba48ec08cddd04c8c40215058721/Actors%3A%20A%20Model%20of%20Concurrent%20Computation%20in%20Distributed%20Systems..md#actors-a-model-of-concurrent-computation-in-distributed-systems) <br>
Gul Agha, 1985 <br>
