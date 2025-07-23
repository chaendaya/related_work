# A Unified Semantics for Future Erlang



### Abstract

**Erlang의 형식적 의미론(formal semantics)은 이해하기에는 다소 복잡**하다. 이러한 복잡성의 상당 부분은 현재의 구현(Erlang/OTP R11–R14)을 정확하게 모델링하려는 시도에서 비롯된다. 이 구현들은 20년이 넘는 기간 동안 개발된 다양한 기능과 최적화를 포함하고 있다. 그 결과, 시스템, 특히 메시지의 동작이 로컬(local) 환경과 분산(distributed) 환경에서 다르게 작동하는 이중 계층(two-tier) 의미론이 형성되었다. 하지만 멀티코어 하드웨어의 등장과 다중 실행 큐(multiple run-queues), 그리고 효율적인 SMP(Symmetric Multi-Processing) 지원의 도입으로 인해, **로컬과 분산의 경계는 흐려졌으며, 궁극적으로는 이러한 경계를 제거해야 한다.** 본 논문에서는 미래의 Erlang 구현을 위한 훨씬 더 깔끔한 의미론을 새롭게 제시한다. 우리는 이 논문이 현재 및 미래의 Erlang 구현에서 충분히 이해되지 않은 여러 기능들에 대해 활발한 논의가 이루어지는 계기가 되기를 기대한다.



### 1.Introduction

지난 수년간 Erlang의 의미론을 설명하고 정의하려는 많은 노력이 이루어져 왔다. Petterson의 초기 작업 [5], Fredlund의 단일 노드 의미론에 대한 형식적 정의 [3], Armstrong의 고수준 언어 설계 철학 설명 [1], 그리고 Claessen과 Svensson의 분산 Erlang 의미론 정의 및 이를 Svensson과 Fredlund가 다듬은 작업 [6]까지 이어져 왔다. 그 최종 결과는 현재 (Erlang/OTP R11–R14) 구현이 어떻게 동작하는지를 자세히 설명하는 정확한 형식적 의미론이다. Fredlund과 Svensson이 개발한 McErlang의 성공적인 구현 [4]은 이 의미론이 실제로 유용하며 실용적인 목적으로도 사용될 수 있음을 보여준다. 하지만 이 형식적 의미론은 이해하기엔 다소 복잡하다. 이는 언어 설계 철학 [1]이 매우 간결하고 이해하기 쉬운 것과는 대조적이어서 다소 불편한 점이다. 현재 Erlang 의미론을 분석해보면, 그 복잡성의 대부분은 Erlang 런타임 시스템(ERTS)의 현재 구현을 매우 밀접하게 따르려는 시도에서 비롯됨을 알 수 있다.

그 결과, **시스템 ― 특히 메시지 ― 이 로컬 환경과 분산 환경에서 다르게 동작**하는 **이중 계층(two-tier)의 의미론**이 탄생했다. 또한, Erlang 언어는 수년에 걸쳐 발전해 왔으며, 현재 다시 설계한다면 피할 수 있었을 레거시 구조들이 남아 있다. 가장 두드러진 예는 monitor와 link의 중복으로, trap된 링크 메시지와 모니터 메시지 사이에 실제로는 큰 차이가 없다는 점이다.

현재의 추세는 멀티코어 하드웨어의 도입과, 다중 실행 큐 및 효율적인 SMP(Symmetric Multi-Processing) 지원을 포함한 ERTS의 최신 발전이다. 이로 인해 **로컬과 분산의 경계는 모호해졌으며, 궁극적으로는 이러한 경계를 제거해야 한다.**
본 논문에서는 로컬과 원격 프로세스 사이에 경계가 없고, 의미론 자체가 병렬화(parallelization)를 방해하지 않는, 미래의 Erlang 구현을 위한 훨씬 더 깔끔한 새로운 의미론을 제안한다.

우리는 이 논문에서 Erlang의 미래 의미론을 최종적으로 정의하려는 것은 아니며, 이 주제에 대한 논의와 토론이 활발히 시작되기를 바라는 마음으로 글을 쓴다. 우리는 Erlang이 향후 멀티코어 아키텍처로 확장되기 위해서는 현재의 몇 가지 제약을 해소하는 것이 필수적이라 믿는다. 물론, 성숙한 프로그래밍 언어의 의미론을 (급진적으로) 바꾸는 데 수반되는 모든 어려움 ― 예를 들어, 방대한 레거시 코드와 하위 호환성 문제 ― 을 잘 인식하고 있다. 그럼에도 불구하고, 변화를 고민하기 시작하는 시점이 빠를수록, 현실이 따라오기 전에 더 많은 시간을 갖고 준비할 수 있다.

논문 구성
본 논문은 다음과 같이 구성되어 있다.
2장에서는 새로운 의미론에 대해 직관적인 고수준 개요를 먼저 제시하고,
3장에서는 새로운 의미론의 정의와 규칙들을 형식적으로 명시한다.
4장에서는 노드 컨트롤러의 내부 작동 방식을 설명하고,
5장에서는 공정성(fairness)을 위한 실행 제약 규칙을 기술한다.
6장에서는 현재 의미론과 새 의미론 사이의 비교 예시를 몇 가지 제시하고,
마지막으로 7장에서 결론을 내리고 향후 방향을 제시한다.



### 2.직관적인 의미론 (Intuitive Semantics)

정의와 의미론 규칙들을 나열하기에 앞서, 
이 장에서는 새로운 의미론에 대한 비공식적이지만 직관적인 고수준 개요를 먼저 제시한다.
우리의 설정에서, **하나의 완전한 분산 시스템은 여러 개의 노드로 구성**된다. 
이 노드들은 시스템에서 최상위 컨테이너(top-level container) 역할을 한다. 
**각 노드는 노드 컨트롤러(node controller)(이것은 이번 새로운 의미론에서 추가된 개념)와** 
**여러 개의 프로세스를 포함**한다. 
여기서 주의할 점은 노드가 물리적인 머신과 일치하지 않으며, 
오히려 하나의 머신에서 여러 개의 노드가 실행될 수 있다는 점이다. 
이러한 고수준의 분산 시스템 구조는 그림 1(Figure 1)에 나타나 있다.

<img width="567" height="412" alt="Image" src="https://github.com/user-attachments/assets/00c92147-43bd-4186-9f0c-f3ed5ae5b8fd" />


#### 2.1 모든 것은 분산이다 (Everything is Distributed)

서론에서 언급했듯이, 멀티코어 아키텍처의 등장으로 인해 로컬과 원격의 경계는 더욱 모호해졌다. 따라서 새로운 의미론에서는 **자기 자신에게 보내는 메시지를 포함한 모든 메시지를 동일하게 처리**하며, **모든 메시지는 (가상적인) 시스템 메시지 큐(ether)를 통해 전달된다**.

실제로 이는 **모든 메시지 전송이 두 단계로 구성됨을 의미**한다: 전송(sending)과 전달(delivery). 이로 인해 서로 다른 프로세스 간의 메시지들은 자유롭게 재정렬될 수 있다.

또한 우리는, **새로운 프로세스 생성**, **링크 생성(linking)** 등과 같은 대부분의 **부수 효과(side effect) 연산을 비동기적으로 처리**하기로 결정했다. 이러한 다양한 부수 효과들이 시스템에 유사한 영향을 주는 점을 강조하기 위해, 우리는 이를 **노드 컨트롤러**라는 공통 구조를 통해 **일관되게 처리**한다. 이 노드 컨트롤러는 **노드 내의 관리 기능 전체를 담당**한다.

예를 들어, 어떤 프로세스에 대해 이름을 등록하는 것은 **노드 컨트롤러에 신호를 보내는 방식**으로 수행된다. 어느 시점에 노드 컨트롤러는 이 신호를 수신하고, 해당 이름을 등록할 수 있는지를 결정한 후, 등록을 요청한 프로세스에 **응답을 보낸다**. 사용자 입장에서 이것이 다소 비실용적으로 보일 수는 있지만, `register` 신호를 보내고 응답을 기다리는 **고수준 함수**를 정의하는 것은 얼마든지 가능하다.



#### 2.2 단방향 링크만 존재 (Uni-directional Links Only)

새로운 의미론에서도 **링크(link)**와 **모니터(monitor)** 개념은 여전히 존재한다. 하지만 기존 의미론에 있었던 `trap_exit` 기능은 존재하지 않는다. 이는 곧, **링크된 프로세스 중 하나가 종료되면, 연결된 프로세스도 반드시 종료**된다는 의미이며, 어떤 조건이나 예외도 없다. (`trap_exit` 기능은 실제로는 `monitor`와 거의 동일한 역할을 하므로 제거되었다.)

또한, 우리는 **링크와 모니터를 모두 단방향(unidirectional)으로 처리하기로 결정**했다. 즉, **프로세스 A가 프로세스 B를 링크(link)하거나 모니터(monitor)할 경우, A의 실패는 B에 아무 영향도 주지 않는다.**

다음은 단방향 링크와 모니터를 사용하여 supervisor 동작을 재구현하는 예제이다.
 자식 프로세스를 `{M, F, A}` 튜플로 정의하고, 자식이 종료되면 supervisor가 이를 감지하며, 반대로 supervisor가 비정상적으로 종료되면 자식도 종료되도록 구현하려면, 아래 코드 조각으로 충분하다:

````
SupervisorPid = self(),
ChildPid = spawn(fun() ->
    link(SupervisorPid),
    apply(M, F, A)
end),
MonitorRef = monitor(process, ChildPid),
````



#### 2.3 내장된 프로세스 레지스트리 (A Built-in Registry)

우리는 새로운 의미론에 **프로세스 레지스트리(process registry)**를 **포함시키는 것을 선택**했다. 이 선택이 논란의 여지가 있을 수는 있으나, **이름 있는 프로세스(named process)** 간의 통신은 Erlang에서 매우 핵심적인 개념이므로, 의미론 내에 포함할 가치가 있다고 판단했다.

기존 Erlang과 마찬가지로, 지원되는 기본 연산은 다음과 같다:

- **이름 있는 프로세스에 메시지 전송**: `atom ! Msg`
- **원격 노드에 있는 이름 있는 프로세스에 메시지 전송**: `{atom, node} ! Msg`
- **이름 등록(register)**, **해제(unregister)**, **이름 조회(whereis)**

하지만 일관성을 위해, 이 의미론에서는 다음과 같은 동작도 허용된다:

- **원격 프로세스에 이름 등록**: `register(Name, Pid)`는 `Pid`가 원격 프로세스여도 실패하지 않음
- **로컬 프로세스를 원격 노드에 이름 등록**: `register(Node, Name, Pid)`를 통해 가능

이러한 설계의 결과로, `{atom, node} ! Msg` 문법을 사용해 메시지를 전송할 경우, **그 메시지를 수신해야 하는 프로세스가 실제로 그 노드에 있을 것이라는 보장이 없다**. 따라서 **다른 노드로 메시지를 중계(relay)** 해야 할 수도 있다.





#### 2.4 메시지 전달에 대한 보장 (Message-Passing Guarantees)

새로운 의미론에서는 **일반적으로 메시지 전달에 대한 보장이 거의 없다.** 하지만 각 프로세스 쌍(process pair) 사이에서는 메시지 순서(order)가 보장된다. 즉, 프로세스 A가 프로세스 B에게 메시지 스트림 `M1; M2; M3; ...`를 전송하면, 프로세스 B는 정확히 그 순서대로 메시지를 수신하게 된다.
 (단, 노드 연결 해제(node disconnect)로 인해 시퀀스의 마지막 쪽 메시지들이 누락(dropped)될 가능성은 존재한다.)

이러한 메시지 순서 보장은 Armstrong의 박사논문 [1]에서 설명된 바와 정확히 일치한다. 그러나 Svensson과 Fredlund [7]이 지적했듯이, 현재의 Erlang 구현에서는 이러한 보장이 제공되지 않는다. 특히 분산된 프로세스들 사이에서 통신이 이루어질 경우, 메시지가 손실될 수 있다.

또한, 의미론 상 **다음과 같은 경우 메시지 순서에 대한 보장은 존재하지 않는다**:

- **직접 주소 지정**(예: `Pid ! Msg`)과
- **이름을 통한 간접 주소 지정**(예: `RegName ! Msg`)이 **동시에 사용된 경우**

예를 들어, 프로세스 `P`가 다음과 같은 코드 조각을 실행한다고 가정하자:

```Q ! msg1, q_name ! msg2
Q ! msg1, q_name ! msg2
```

여기서 `Q`는 어떤 프로세스의 `pid`이고, 그 프로세스는 `q_name`이라는 이름으로도 등록되어 있다고 하자.
이 경우, `msg1`과 `msg2` 중 어떤 메시지가 먼저 Q의 메일박스(mailbox)에 도달할지는 보장되지 않는다.







### 3. 형식 의미론 (Formal Semantics)

이 장에서는 이전 Erlang 형식 의미론들과 유사한 스타일로 새로운 의미론을 제시한다.
 이 스타일은 직관적이고 따라가기 쉬우며, 동시에 정확성에 대한 엄밀한 논의를 가능하게 할 정도로 충분히 상세하다.
 우리는 의미론 규칙을 제시하기에 앞서 필요한 정의들을 먼저 제시한다.



##### Definition 1.

**프로세스**는 `p ∈ Process`로 표현되며, 다음과 같은 삼중항(triplet)으로 구성된다:
```math
(e, \mathit{pid}, q)
```
여기서:

- `e`는 현재 프로세스가 실행 중인 **표현식(expression)**
- `pid`는 **프로세스 식별자(process identifier)**
- `q`는 **메시지 큐(message queue)**

이때 `e`는 일반적인 Erlang 표현식으로, [3]에서 정의된 표현식들과 유사하게 해석된다.



##### Definition 2.

**프로세스 그룹**은 `pg ∈ ProcessGroup`로 표현되며 다음 셋 중 하나이다:

- **빈 프로세스 그룹** : <img width="18" height="23" alt="Image" src="https://github.com/user-attachments/assets/49b49030-4cbe-4eed-afec-6c941f2aca40" />
- **단일 프로세스**
- **두 프로세스 그룹의 결합** : <img width="113" height="31" alt="Image" src="https://github.com/user-attachments/assets/e088e5d2-442a-452a-a59f-ac1c3741a6ac" />



##### Definition 3.

**노드 컨트롤러(node controller)**는 `nc ∈ NodeController`로 표현되며 다음과 같은 삼중항이다:
```math
(lnks, mns, reg)
```
여기서:

- `lnks`: **링크 집합**, 각 요소는 `(link_from, link_to)` 형태의 튜플
             `P(ProcessIdentier X ProcessIdentier)`
- `mns`: **모니터 목록**, 각 요소는 `(mon_name, mon_from, mon_to)` 형태의 튜플
           ` P(MonitorReference X ProcessIdentier X ProcessIdentier)`
- `reg`: **등록된 이름의 집합**, 각 요소는 `(name, pid)` 형태의 튜플
            `P(ProcessIdentier X ProcessName)`

즉, 노드 컨트롤러는 다음 세 가지 정보를 포함한다:

1. 프로세스 간의 링크 정보
2. 모니터 설정 정보
3. 프로세스 이름 등록 상태



##### Definition 4.

**시스템 메시지 큐(system message queue)**, 즉 **ether**는 `eth ∈ SystemMessageQueue`로 표현되며, 다음과 같은 유한한 삼중항의 시퀀스로 구성된다:
```math
(id_{from}, id_{to}, signal)
```

- `Identifier`는 프로세스 식별자(`ProcessIdentifier`)와 노드 식별자(`NodeIdentifier`)의 합집합이며, 이를 `id ∈ Identifier`로 표현한다.
- `signal`은 전달되는 신호를 의미한다.
- `e`는 **빈 시퀀스(empty sequence)**를 의미하며, <img width="22" height="18" alt="Image" src="https://github.com/user-attachments/assets/ff9f07fd-ae36-4600-84f7-b489255d70fe" />
   `.`은 시퀀스의 **연결(concatenation)**,
   `\`는 **첫 번째로 일치하는 항목을 삭제**하는 연산이다.

예시

```eth = (a2, b1, c1).(a1, b2, c1).(a1, b2, c2).(a1, b2, c1)
eth = (a2, b1, c1).(a1, b2, c1).(a1, b2, c2).(a1, b2, c1)
eth \ (a1, b2, c1) = (a2, b1, c1).(a1, b2, c2).(a1, b2, c1)
```



##### Definition 5.

**노드(node)**는 `n ∈ Node`로 표현되며, 다음과 같은 삼중항이다:
```math
[pg, nid, nc]
```
여기서:

- `pg`는 해당 노드에서 실행 중인 **프로세스 그룹**
- `nid`는 노드의 **고유 식별자(node identifier)**
- `nc`는 **노드 컨트롤러**

즉, 하나의 노드는 프로세스 그룹, 노드 ID, 노드 컨트롤러 세 가지로 구성된다.



##### Definition 6.

노드 시스템(node system)은 `ns ∈ P Node`로 표현되며 다음 중 하나이다:

- **빈 노드 시스템**: <img width="18" height="23" alt="Image" src="https://github.com/user-attachments/assets/49b49030-4cbe-4eed-afec-6c941f2aca40" />
- **단일 노드**
- **두 노드 시스템의 결합**: <img width="98" height="23" alt="Image" src="https://github.com/user-attachments/assets/46d57e56-88c9-48de-934e-d0f73b70f536" />



##### Definition 7.

시스템은 `s ∈ System`으로 표현되며, 다음과 같은 쌍(tuple)으로 구성된다:
```math
[[ns, eth]]
```

- `ns`는 **노드 시스템(node system)**
- `eth`는 **에테르(ether, 즉 시스템 메시지 큐)**

직관적으로 보면, 프로세스들을 프로세스 그룹으로, 노드들을 노드 시스템으로 구성하는 것은 단순히 **프로세스(또는 노드)**의 **집합(set)**으로 생각할 수 있다. 우리는 의미론 정의에서 <img width="26" height="32" alt="Image" src="https://github.com/user-attachments/assets/ef535832-cc43-4e06-b44f-6dd71e73bf41" /> (프로세스 그룹 병합 연산자)와 <img width="23" height="28" alt="Image" src="https://github.com/user-attachments/assets/fff7c16d-f358-49ea-a778-556706953855" /> (노드 시스템 병합 연산자)가 **결합법칙(associative)**과 **교환법칙(commutative)**을 만족하도록 조심스럽게 설계할 것이다.

또한, 의미론 규칙을 간결하고 읽기 쉽게 유지하기 위해 **자주 반복되는 복잡한 표현**을 줄이기 위한 **보조 함수들(supportive functions)**을 정의한다.



##### Definition 8.

함수 `isNid(i)`는, 어떤 식별자 `i ∈ Identifier`가 **노드 식별자**인 경우에는 `true`를, **프로세스 식별자**인 경우에는 `false`를 반환한다.



##### Definition 9.

함수 `node(p)`는, 프로세스 식별자 `p ∈ ProcessIdentifier`에 대해, 그 프로세스가 속한 **노드 식별자(node identifier)**를 반환한다.



##### Definition 10.

함수 `destNid(sig)`는 어떤 **신호(signal)** `sig`에 관련된 **원격 노드의 식별자**를 반환한다. 일부 신호에 대해서는 **정의되지 않을 수도 있음(undefined)**에 주의하십시오.

예시:

- `destNid(link(pid))` → `node(pid)`
- `destNid(spawn(e, ref))` → `undefined`



##### Definition 11.

함수 `ethMatch(eth, to, from)`는 **시스템 메시지 큐(eth)**에서, **보낸 주체(from)**가 **수신자(to)**에게 보낸 **첫 번째 메시지**를 반환한다.

예시:

```
eth = (a2, b1, c1).(a1, b2, c1).(a1, b2, c2).(a1, b2, c1)
ethMatch(eth, a1, b2) = c1
```



##### Definition 12.

- 함수 `pids(pg)`는 프로세스 그룹 `pg` 내의 **모든 프로세스 식별자(pid)**들의 집합을 반환한다.
- 함수 `nids(ns)`는 노드 시스템 `ns` 내의 **모든 노드 식별자(nid)**들의 집합을 반환한다.

또한 다음 조건을 만족할 때 프로세스 그룹 또는 시스템을 **"정상적(well-formed)"**이라고 한다:

- 각 **프로세스 식별자**는 **하나의 프로세스에만 속함**
- 각 **노드 식별자**는 **하나의 노드에만 속함**
- 하나의 식별자는 오직 **노드 식별자이거나 프로세스 식별자 중 하나**이어야 함



이후의 의미론에서는 모든 Erlang 노드 시스템이 **정상적(well-formed)**이고, 포함된 프로세스 그룹 역시 **정상적인 구조**를 갖는다고 가정한다.

의미론에서 **신호(signal)**는 **송신자 프로세스**와 **수신자 프로세스(또는 노드 컨트롤러)** 간에 전달되는 **정보 항목**이다.

**프로세스의 동작(process action)**은 다음 중 하나이다:

- 침묵 동작(silent action)
- 입력 동작(input action)
- 출력 동작(output action
- 노드 종료(node termination)
- 노드 연결 해제(node disconnect)



##### Definition 13. (Process signals)

프로세스 신호는 `sig ∈ Signal`로 범위가 정해지며, 다음과 같은 형식을 갖는다:

- `message(v)`                    메시지 `v`를 전달하는 일반적인 메시지 전송
- `link(pid)`                     프로세스 `pid`와 링크(link) 설정
- `unlink(pid)`                   프로세스 `pid`와 링크 해제
- `monitor(pid, ref)`             프로세스 `pid`를 모니터링하며 참조 ID는 `ref`
- `unmonitor(ref)`                모니터링을 중단 (참조 ID `ref`)
- `monitor_node(nid)`             노드 `nid`를 모니터링
- `unmonitor_node(ref)`           노드 모니터링 해제 (참조 ID `ref`)
- `whereis(name)`                 이름 `name`으로 등록된 프로세스 ID(pid)를 조회
- `register(name, pid)`           이름 `name`에 프로세스 ID `pid`를 등록
- `spawn(e, ref)`                 표현식 `e`를 통해 프로세스를 생성하며 참조 ID는 `ref`
- `spawn_node()`                  새로운 노드를 생성
- `nsend(name, v)`                이름 `name`에 바인딩된 프로세스에 메시지 `v`를 전송 (named send)
- `exit(v)`                       외부 종료(exit) 신호 전달
- `died(id, v)`                   프로세스나 노드의 종료를 알리는 신호 (id가 종료되었으며 값은 `v`)



##### Definition 14. (Process actions)

프로세스 동작은 `α ∈ Action`으로 정의되며, 다음을 포함한다:

- `τ`  내부 계산 (silent action)
- `pid !_from sig`  프로세스 `pid`에게 `from`으로부터 `sig` 신호를 보냄 (출력 동작)
- `pid ?_from sig`  프로세스 `pid`가 `from`으로부터 `sig` 신호를 수신 (입력 동작)
- `die(nid)`  노드 `nid`의 종료
- `disconnect(nid₁, nid₂)`  노드 `nid₁`와 `nid₂` 간 연결 해제



**다음으로, 우리는 Erlang 시스템의 가능한 계산 단계들을 형식적으로 정의한다.**

이 정의에서는 표현식에 대한 전이 규칙 집합이 존재한다고 가정한다. 이 집합은 `P(Expression × exprAction × Expression)`의 부분집합이다. 참조 [3]에 존재한다. 완전성을 위해 아래에 표현식 동작의 정의를 반복한다:



##### Definition 15. (Expression actions)

표현식 동작은 `α ∈ exprAction`으로 정의되며 다음을 포함한다:

- `τ`  계산 단계
- `pid ! v`  프로세스 `pid`에 값 `v`를 전송
- `exiting(v)`  예외 발생 또는 종료
- `read(q, v)`   큐 `q`에서 값 `v`를 읽음
- `test(q)`   큐 `q`의 내용을 검사
- `f(v₁, ..., vₙ) ~> v`   내장 함수 `f`를 인자 `v₁, ..., vₙ`으로 호출하여 결과 `v`를 얻음

직관적으로, `f(v₁, ..., vₙ) ~> v`는 내장 함수 호출을 의미하며, 결과 값은 `v`이다.



##### Definition 16.

함수 `mkAction(msgs)`는 다음과 같이 정의된다:

```
mkAction(ε) = ε
mkAction((to, from, sig).msgs) = to !_from sig ; mkAction(msgs)
```

즉, 메시지 큐를 처리하여 출력 동작들을 생성하는 함수이다.



##### Definition 17.

시스템 전이 관계(system transition relation)는 **표 1 ~ 9**의 전이 규칙을 만족하는 **최소 관계**이다.
