# A Unified Semantics for Future Erlang

<br>
<br>


## Abstract

**Erlang의 형식적 의미론(formal semantics)은 이해하기에는 다소 복잡**하다. 이러한 복잡성의 상당 부분은 현재의 구현(Erlang/OTP R11–R14)을 정확하게 모델링하려는 시도에서 비롯된다. 이 구현들은 20년이 넘는 기간 동안 개발된 다양한 기능과 최적화를 포함하고 있다. 그 결과, 시스템, 특히 메시지의 동작이 로컬(local) 환경과 분산(distributed) 환경에서 다르게 작동하는 이중 계층(two-tier) 의미론이 형성되었다. 하지만 멀티코어 하드웨어의 등장과 다중 실행 큐(multiple run-queues), 그리고 효율적인 SMP(Symmetric Multi-Processing) 지원의 도입으로 인해, **로컬과 분산의 경계는 흐려졌으며, 궁극적으로는 이러한 경계를 제거해야 한다.** 본 논문에서는 미래의 Erlang 구현을 위한 훨씬 더 깔끔한 의미론을 새롭게 제시한다. 우리는 이 논문이 현재 및 미래의 Erlang 구현에서 충분히 이해되지 않은 여러 기능들에 대해 활발한 논의가 이루어지는 계기가 되기를 기대한다.

<br>
<br>

## 1.Introduction

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

<br>
<br>

## 2.직관적인 의미론 (Intuitive Semantics)

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

<br>
<br>

#### 2.1 모든 것은 분산이다 (Everything is Distributed)

서론에서 언급했듯이, 멀티코어 아키텍처의 등장으로 인해 로컬과 원격의 경계는 더욱 모호해졌다. 따라서 새로운 의미론에서는 자기 자신에게 보내는 메시지를 포함한 모든 메시지를 동일하게 처리하며, **모든 메시지는 (가상적인) 시스템 메시지 큐(ether)를 통해 전달된다**.

실제로 이는 **모든 메시지 전송이 두 단계로 구성됨을 의미**한다: 전송(sending)과 전달(delivery). 이로 인해 서로 다른 프로세스 간의 메시지들은 자유롭게 재정렬될 수 있다.

또한 우리는, 새로운 프로세스 생성, 링크 생성(linking) 등과 같은 대부분의 **부수 효과(side effect) 연산을 비동기적으로 처리**하기로 결정했다. 이러한 다양한 부수 효과들이 시스템에 유사한 영향을 주는 점을 강조하기 위해, 우리는 이를 **노드 컨트롤러**라는 공통 구조를 통해 **일관되게 처리**한다. 이 노드 컨트롤러는 노드 내의 관리 기능 전체를 담당한다.

예를 들어, 어떤 프로세스에 대해 이름을 등록하는 것은 **노드 컨트롤러에 신호를 보내는 방식**으로 수행된다. 어느 시점에 노드 컨트롤러는 이 신호를 수신하고, 해당 이름을 등록할 수 있는지를 결정한 후, 등록을 요청한 프로세스에 **응답을 보낸다**. 사용자 입장에서 이것이 다소 비실용적으로 보일 수는 있지만, register 신호를 보내고 응답을 기다리는 고수준 함수를 정의하는 것은 얼마든지 가능하다.

<br>

#### 2.2 단방향 링크만 존재 (Uni-directional Links Only)

새로운 의미론에서도 링크(link)와 모니터(monitor) 개념은 여전히 존재한다. 하지만 기존 의미론에 있었던 trap_exit 기능은 존재하지 않는다. 이는 곧, 링크된 프로세스 중 하나가 종료되면, 연결된 프로세스도 반드시 종료된다는 의미이며, 어떤 조건이나 예외도 없다. (trap_exit 기능은 실제로는 monitor와 거의 동일한 역할을 하므로 제거되었다.)

또한, 우리는 링크와 모니터를 모두 단방향(unidirectional)으로 처리하기로 결정했다. 즉, 프로세스 A가 프로세스 B를 링크(link)하거나 모니터(monitor)할 경우, A의 실패는 B에 아무 영향도 주지 않는다.

다음은 단방향 링크와 모니터를 사용하여 supervisor 동작을 재구현하는 예제이다.
 자식 프로세스를 $`\{M, F, A\}`$ 튜플로 정의하고, 자식이 종료되면 supervisor가 이를 감지하며, 반대로 supervisor가 비정상적으로 종료되면 자식도 종료되도록 구현하려면, 아래 코드 조각으로 충분하다:

````
SupervisorPid = self(),
ChildPid = spawn(fun() ->
    link(SupervisorPid),
    apply(M, F, A)
end),
MonitorRef = monitor(process, ChildPid),
````

<br>

#### 2.3 내장된 프로세스 레지스트리 (A Built-in Registry)

우리는 새로운 의미론에 프로세스 레지스트리(process registry)를 포함시키는 것을 선택했다. 이 선택이 논란의 여지가 있을 수는 있으나, 이름 있는 프로세스(named process) 간의 통신은 Erlang에서 매우 핵심적인 개념이므로, 의미론 내에 포함할 가치가 있다고 판단했다.

기존 Erlang과 마찬가지로, 지원되는 기본 연산은 다음과 같다:

- 이름 있는 프로세스에 메시지 전송: $`atom \ \  ! \ \ Msg`$
- 원격 노드에 있는 이름 있는 프로세스에 메시지 전송: $`\{atom, \ \ node\} \ \ ! \ \ Msg`$
- 이름 등록(register), 해제(unregister), 이름 조회(whereis)

하지만 일관성을 위해, 이 의미론에서는 다음과 같은 동작도 허용된다:

- 원격 프로세스에 이름 등록: $`register(Name, \ \ Pid)`$는 $`Pid`$가 원격 프로세스여도 실패하지 않음
- 로컬 프로세스를 원격 노드에 이름 등록: $`register(Node, \ \ Name, \ \ Pid)`$를 통해 가능

이러한 설계의 결과로, $`\{atom, \ \ node\} \ \ ! \ \ Msg`$ 문법을 사용해 메시지를 전송할 경우, 그 메시지를 수신해야 하는 프로세스가 실제로 그 노드에 있을 것이라는 보장이 없다. 따라서 다른 노드로 메시지를 중계(relay) 해야 할 수도 있다.

<br>

#### 2.4 메시지 전달에 대한 보장 (Message-Passing Guarantees)

새로운 의미론에서는 **일반적으로 메시지 전달에 대한 보장이 거의 없다.** 하지만 각 프로세스 쌍(process pair) 사이에서는 메시지 순서(order)가 보장된다. 즉, 프로세스 A가 프로세스 B에게 메시지 스트림 $`M1; M2; M3; ...`$를 전송하면, 프로세스 B는 정확히 그 순서대로 메시지를 수신하게 된다.
 (단, 노드 연결 해제(node disconnect)로 인해 시퀀스의 마지막 쪽 메시지들이 누락(dropped)될 가능성은 존재한다.)

이러한 메시지 순서 보장은 Armstrong의 박사논문 [1]에서 설명된 바와 정확히 일치한다. 그러나 Svensson과 Fredlund [7]이 지적했듯이, 현재의 Erlang 구현에서는 이러한 보장이 제공되지 않는다. 특히 분산된 프로세스들 사이에서 통신이 이루어질 경우, 메시지가 손실될 수 있다.

또한, 의미론 상 다음과 같은 경우 메시지 순서에 대한 보장은 존재하지 않는다:

- 직접 주소 지정 (예: $`Pid \ \ ! \ \ Msg`$)과
- 이름을 통한 간접 주소 지정 (예: $`RegName \ \ ! \ \ Msg`$)이 동시에 사용된 경우

예를 들어, 프로세스 $`P`$가 다음과 같은 코드 조각을 실행한다고 가정하자:

```
Q ! msg1, q_name ! msg2
Q ! msg1, q_name ! msg2
```

여기서 $`Q`$는 어떤 프로세스의 $`pid`$이고, 그 프로세스는 $`q_{name}`$이라는 이름으로도 등록되어 있다고 하자.
이 경우, $`msg1`$과 $`msg2`$ 중 어떤 메시지가 먼저 Q의 메일박스(mailbox)에 도달할지는 보장되지 않는다.


<br>
<br>


## 3. 형식 의미론 (Formal Semantics)

이 장에서는 이전 Erlang 형식 의미론들과 유사한 스타일로 새로운 의미론을 제시한다.
 이 스타일은 직관적이고 따라가기 쉬우며, 동시에 정확성에 대한 엄밀한 논의를 가능하게 할 정도로 충분히 상세하다.
 우리는 의미론 규칙을 제시하기에 앞서 필요한 정의들을 먼저 제시한다.

<br>

$`\textbf{Definition 1.}\quad`$ **프로세스**는 `p ∈ Process`로 표현되며, 다음과 같은 삼중항(triplet)으로 구성된다:
```math
\langle e, \mathit{pid}, q \rangle
```
여기서:

- $`e`$ 는 현재 프로세스가 실행 중인 표현식(expression)
- $`pid`$ 는 프로세스 식별자(process identifier)
- $`q`$ 는 메시지 큐(message queue)

이때 $`e`$는 일반적인 Erlang 표현식으로, [3]에서 정의된 표현식들과 유사하게 해석된다.

<br>

$`\textbf{Definition 2.}\quad`$ **프로세스 그룹**은 `pg ∈ ProcessGroup`로 표현되며 다음 셋 중 하나이다:

- 빈 프로세스 그룹 : $`\emptyset`$
- 단일 프로세스
- 두 프로세스 그룹의 결합 : $`pg_1 \ \  \Vert_P \ \ pg_2`$

<br>

$`\textbf{Definition 3.}\quad`$ **노드 컨트롤러(node controller)** 는 `nc ∈ NodeController`로 표현되며 다음과 같은 삼중항이다:
```math
\langle lnks, mns, reg \rangle
```
여기서:

- $`lnks`$ : 링크 집합, 각 요소는 $`(link\_from, link\_to)`$ 형태의 튜플 $` \qquad\qquad\qquad\quad\quad`$ $`\mathcal{P}`$(ProcessIdentier $`\times`$ ProcessIdentier)
- $`mns`$ : 모니터 목록, 각 요소는 $`(mon\_name, mon\_from, mon\_to)`$ 형태의 튜플 $` \quad\quad`$ $`\mathcal{P}`$(MonitorReference $`\times`$ ProcessIdentier $`\times`$ ProcessIdentier)
- $`reg`$ : 등록된 이름의 집합, 각 요소는 $`(name, pid)`$ 형태의 튜플 $` \qquad\qquad\qquad\quad\quad`$ $`\mathcal{P}`$(ProcessIdentier $`\times`$ ProcessName)

즉, 노드 컨트롤러는 다음 세 가지 정보를 포함한다:

1. 프로세스 간의 링크 정보
2. 모니터 설정 정보
3. 프로세스 이름 등록 상태

<br>

$`\textbf{Definition 4.}\quad`$ **시스템 메시지 큐(system message queue)**, <br> <br>
즉 **ether**는 `eth ∈ SystemMessageQueue`로 표현되며, 다음과 같은 유한한 삼중항의 시퀀스로 구성된다:
```math
\langle id_{from}, id_{to}, signal \rangle
```
<br>

- $`Identifier`$ 는 프로세스 식별자(ProcessIdentifier)와 노드 식별자(NodeIdentifier)의 합집합이며, 이를 `id ∈ Identifier`로 표현한다.
- $`signal`$ 은 전달되는 신호를 의미한다.
- $`\emptyset`$는 빈 시퀀스(empty sequence)를 의미하며,
   $`.`$은 시퀀스의 연결(concatenation),
   $`\setminus`$는 첫 번째로 일치하는 항목을 삭제하는 연산이다.

예시:

```math
eth = (a2, b1, c1) \cdot (a1, b2, c1) \cdot (a1, b2, c2) \cdot (a1, b2, c1)
```
```math
eth \setminus (a1, b2, c1) = (a2, b1, c1) \cdot (a1, b2, c2) \cdot (a1, b2, c1)
```
<br>
<br>

$`\textbf{Definition 5.}\quad`$ **노드(node)** 는 `n ∈ Node`로 표현되며, 다음과 같은 삼중항이다:
```math
[pg, nid, nc]
```
여기서:

- $`pg`$는 해당 노드에서 실행 중인 프로세스 그룹
- $`nid`$는 노드의 고유 식별자(node identifier)
- $`nc`$는 노드 컨트롤러

즉, 하나의 노드는 프로세스 그룹, 노드 ID, 노드 컨트롤러 세 가지로 구성된다.

<br>

$`\textbf{Definition 6.}\quad`$ **노드 시스템(node system)** 은 `ns ∈ P Node`로 표현되며 다음 중 하나이다:

- 빈 노드 시스템: $`\emptyset`$
- 단일 노드
- 두 노드 시스템의 결합:  $`ns_1 \ \  \Vert_N \ \ ns_2`$

<br>

$`\textbf{Definition 7.}\quad`$ **시스템** 은 `s ∈ System`으로 표현되며, 다음과 같은 쌍(tuple)으로 구성된다:
```math
[[ ns, eth ]]
```

- $`ns`$ 는 노드 시스템(node system)
- $`eth`$ 는 에테르(ether, 즉 시스템 메시지 큐)

직관적으로 보면, 프로세스들을 프로세스 그룹으로, 노드들을 노드 시스템으로 구성하는 것은 단순히 프로세스(또는 노드)의 집합(set)으로 생각할 수 있다. <br>
우리는 의미론 정의에서 $`pg_1 \ \  \Vert_P \ \ pg_2`$ (프로세스 그룹 병합 연산자)와 $`ns_1 \ \  \Vert_N \ \ ns_2`$ (노드 시스템 병합 연산자)가 결합법칙(associative)과 교환법칙(commutative)을 만족하도록 조심스럽게 설계할 것이다.

<br>

또한, 의미론 규칙을 간결하고 읽기 쉽게 유지하기 위해 자주 반복되는 복잡한 표현을 줄이기 위한 보조 함수들(supportive functions)을 정의한다.


$`\textbf{Definition 8.}\quad`$ 함수 **isNid(i)** 는, 어떤 식별자 `i ∈ Identifier`가 노드 식별자인 경우에는 true를, 프로세스 식별자인 경우에는 false를 반환한다.

<br>

$`\textbf{Definition 9.}\quad`$ 함수 **node(p)** 는, 프로세스 식별자 `p ∈ ProcessIdentifier`에 대해, 그 프로세스가 속한 노드 식별자(node identifier)를 반환한다.

<br>

$`\textbf{Definition 10.}\quad`$ 함수 **destNid(sig)** 는 어떤 신호(signal) $`sig`$에 관련된 원격 노드의 식별자를 반환한다. <br>
일부 신호에 대해서는 정의되지 않을 수도 있음(undefined)에 주의하십시오.

예시:

- $`\textit{destNid(link}(pid)) \rightarrow \textit{node}(pid)`$
- $`\textit{destNid(spawn}(e, ref)) → undefined`$

<br>

$`\textbf{Definition 11.}\quad`$ 함수 **ethMatch(eth, to, from)** 는 시스템 메시지 큐(eth)에서, 보낸 주체(from)가 수신자(to)에게 보낸 첫 번째 메시지를 반환한다.

예시:
```math
eth = (a2, b1, c1) \cdot (a1, b2, c1) \cdot (a1, b2, c2) \cdot (a1, b2, c1)
```
```math
ethMatch(eth, a1, b2) = c1
```
<br>

$`\textbf{Definition 12.}\quad`$

- 함수 $`pids(pg)`$는 프로세스 그룹 $`pg`$ 내의 모든 프로세스 식별자(pid)들의 집합을 반환한다.
- 함수 $`nids(ns)`$는 노드 시스템 $`ns`$ 내의 모든 노드 식별자(nid)들의 집합을 반환한다.

<br>

또한 다음 조건을 만족할 때 프로세스 그룹 또는 시스템을 "정상적(well-formed)"이라고 한다:

- 각 프로세스 식별자는 하나의 프로세스에만 속함
- 각 노드 식별자는 하나의 노드에만 속함
- 하나의 식별자는 오직 노드 식별자이거나 프로세스 식별자 중 하나이어야 함

이후의 의미론에서는 모든 Erlang 노드 시스템이 정상적(well-formed)이고, 포함된 프로세스 그룹 역시 정상적인 구조를 갖는다고 가정한다.

<br>

$`\textbf{Definition 13.}\quad`$ **(Process Signal)** 프로세스 신호는 `sig ∈ Signal`로 범위가 정해지며, 다음과 같은 형식을 갖는다:

<img width="548" height="330" alt="Image" src="https://github.com/user-attachments/assets/999304a3-4810-4694-a655-ac072b3639e5" />


- 메시지 $`v`$ 를 전달하는 일반적인 메시지 전송
- 프로세스 $`pid`$ 와 링크(link) 설정
- 프로세스 $`pid`$ 와 링크 해제
- 프로세스 $`pid`$ 를 모니터링하며 참조 ID는 $`ref`$
- 모니터링을 중단 (참조 ID $`ref`$)
- 노드 $`nid`$ 를 모니터링
- 노드 모니터링 해제 (참조 ID $`ref`$)
- 이름 $`name`$ 으로 등록된 프로세스 ID(pid)를 조회
- 이름 $`name`$ 에 프로세스 ID $`pid`$ 를 등록
- 표현식 $`e`$ 를 통해 프로세스를 생성하며 참조 ID는 $`ref`$
- 새로운 노드를 생성
- 이름 $`name`$ 에 바인딩된 프로세스에 메시지 $`v`$ 를 전송 (named send)
- 외부 종료(exit) 신호 전달
- 프로세스나 노드의 종료를 알리는 신호 (id가 종료되었으며 값은 $`v`$)

의미론에서 **신호(signal)** 는 송신자 프로세스와 수신자 프로세스(또는 노드 컨트롤러) 간에 전달되는 정보 항목이다.

<br>

$`\textbf{Definition 14.}\quad`$ **(Process actions)** `α ∈ Action` 으로 표현되며, 프로세스의 동작은 다음 중 하나이다:

<img width="494" height="121" alt="Image" src="https://github.com/user-attachments/assets/b26904f5-6677-4b72-9b82-e97a23eba72a" />

- $`τ`$  내부 계산 (silent action)
- $`pid \ \ !_{from} \ \ sig`$  프로세스 $`pid`$ 에게 $`from`$ 으로부터 $`sig`$ 신호를 보냄 (출력 동작)
- $`pid \ \ ?_{from} \ \ sig`$  프로세스 $`pid`$ 가 $`from`$ 으로부터 $`sig`$ 신호를 수신 (입력 동작)
- $`die(nid)`$  노드 $`nid`$ 의 종료
- $`disconnect(nid_1, nid_2)`$  노드 $`nid_1`$ 와 $`nid_2`$ 간 연결 해제

<br>

다음으로, 우리는 Erlang 시스템의 가능한 계산 단계들을 형식적으로 정의한다.

이 정의에서는 표현식에 대한 전이 규칙 집합이 존재한다고 가정한다. 이 집합은 P(Expression × exprAction × Expression)의 부분집합이다. <br>
참조 [3]에 존재한다. 완전성을 위해 아래에 표현식 동작의 정의를 반복한다:

<br>

$`\textbf{Definition 15.}\quad`$ **(Expression actions)** 표현식 동작은 `α ∈ exprAction`으로 정의되며 다음을 포함한다:

<img width="506" height="149" alt="Image" src="https://github.com/user-attachments/assets/f0d11fec-a914-4e6a-8771-147195bf3771" />

- $`τ \ \ \ \ \ \ \ \ \ \ \ \ \ \  \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ `$ 계산 단계
- $`pid \ \  !  \ \ v \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ `$ 프로세스 $`pid`$ 에 값 $`v`$ 를 전송
- $`exiting(v) \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ `$ 예외 발생 또는 종료
- $`read(q, v) \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ `$ 큐 $`q`$ 에서 값 $`v`$ 를 읽음
- $`test(q) \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ `$ 큐 $`q`$ 의 내용을 검사
- $`f(v_1, ..., v_n) \leadsto v \ \ \ `$   내장 함수 $`f`$ 를 인자 $`v_1, ..., v_n`$ 으로 호출하여 결과 $`v`$ 를 얻음

직관적으로, $`f(v_1, ..., v_n) \rightsquigarrow v`$ 는 내장 함수 호출을 의미하며, 결과 값은 $`v`$ 이다.

<br>

$`\textbf{Definition 16.}\quad`$ 함수 **mkAction(msgs)** 는 다음과 같이 정의된다:

$` mkAction(\epsilon) = \epsilon `$ <br>
$` mkAction((to, from, sig) \cdot msgs) = to  \ \ !_{from} \ \ sig ; mkAction(msgs) `$

즉, 메시지 큐를 처리하여 출력 동작들을 생성하는 함수이다.

<br>

$`\textbf{Definition 17.}\quad`$ **시스템 전이 관계(system transition relation)** 는 표 1 ~ 9의 전이 규칙을 만족하는 최소 관계이다.

<br>

어떤 규칙들은 원하는 효과를 얻기 위해 함께 결합되어야 한다는 점을 지적할 필요가 있다. 가장 명확한 예는 아마도 출력 관련 규칙들일 것이다. 이 경우, 메시지는 Table 2의 부수 효과(side eff) 규칙에서 전송되고, Table 5의 출력(output) 규칙에서 전달된다. 대부분의 의미론 규칙들은 자명하지만, 우리는 아래에서 각 테이블을 살펴보며 몇 가지 미묘한 점들을 지적하고, 보다 복잡한 규칙들을 설명하고자 한다.

<br>

**프로세스 문맥에서의 표현식 평가** Table 1은 프로세스에 국한된 로컬 표현식 평가를 포함한다. 이 모든 동작들은 더 큰 문맥(즉, 시스템 전체) 안에서 일어나지만, 이 규칙들이 프로세스 외부의 어떤 것에도 의존하지 않기 때문에, 그 문맥은 규칙 내에서 드러나지 않는다. (이러한 규칙들은 Table 5에 정의된 internal-rule에 의해 시스템 수준으로 확장된다.)

<img width="408" height="297" alt="Image" src="https://github.com/user-attachments/assets/62bfbf66-5f67-4d2a-b40e-927fb110963a" />

<img width="557" height="63" alt="image" src="https://github.com/user-attachments/assets/ce026a04-32c9-4e14-818c-55a6d23982e0" />

<br>
<br>

규칙 silent에서는, 표현식 $`e`$ 가 $`e \stackrel{\tau}{\rightarrow} e' `$ 라는 전이(즉, 일반적인 계산 단계)를 가질 경우, 프로세스(그리고 더 크게는 시스템 전체) $`\langle e, pid, q \rangle`$ 는 $`\tau`$ 로 라벨링된 전이를 통해 $` \langle e', pid, q \rangle `$ 로 전이할 수 있다.

read 규칙에서는, 프로세스의 메일박스(큐)가 $` q1 \cdot v \cdot q2 `$ 라는 세 부분으로 나눌 수 있을 때, $` \langle e, \ \ pid, \ \ q1 \cdot v \cdot q2 \rangle `$ 에서 $`\langle e', \ \ pid, \ \ q1 \cdot q2 \rangle`$로 전이하는 것이 가능하다. 
이때 표현식 수준에서 $` receive: \ \  e \stackrel{read(q1, v)}{\longrightarrow} e' `$라는 전이가 유도 가능해야 한다. (정확한 receive 규칙 정의는 [3]을 참조하라.)

따라서 read와 receive 규칙은 함께 receive 구문의 직관적인 의미론을 보장한다. 부수 효과(side effect)를 유발하는 동작은 Table 2에 있는 규칙들에 의해 처리된다.


<img width="448" height="137" alt="image" src="https://github.com/user-attachments/assets/2504ec09-ecbb-483c-b245-3ae217f3a305" />
<br>
<br>

**Side effect를 일으키는 (노드 컨트롤러 관련) 표현식 평가** Table 2는 사이드 이펙트를 발생시키는 동작들에 대한 규칙을 포함한다. 이러한 동작들 중 다수는 노드 컨트롤러의 개입이 필요하다. side eff 규칙은 일반적인(generic) 형태의 규칙으로서 다음과 같은 액션들을 포함한다: send, exit, link, unlink, monitor, unmonitor, spawn, register, whereis, monitor_node, unmonitor_node.

이 테이블의 모든 규칙은 **비동기적(asynchronous)** 이다. 예를 들어 whereis()는 즉시 ok 응답을 반환하지만, "진짜" 결과(예: whereis나 unlink의 경우)는 **나중에 일반 수신 가능한 신호(signal)** 로 도착한다. 이러한 연산들의 비동기적 성격 때문에, 그 의미론 규칙들은 겉보기에는 단순하게 보인다. Table 7에서는 이러한 규칙들에 대해 노드 컨트롤러 측에서 처리되는 부분을 제시하고 있으며, 섹션 5에서는 노드 컨트롤러의 더 복잡한 내부 작동 방식을 설명한다. 

<img width="888" height="253" alt="image" src="https://github.com/user-attachments/assets/bda023bf-f150-4acd-b6ae-b839427fdd9a" />
<br>
<br>

side eff 규칙은 적절한 signal을 구성하기 위해 변환 함수 mkSig를 사용하는데, 이 함수는 Table 3에 정의되어 있다.
<img width="469" height="206" alt="image" src="https://github.com/user-attachments/assets/c524e392-9498-4581-ba9f-1d10835c27ba" />
<br>
<br>

**노드 문맥에서의 표현식 평가** Table 4에는 시스템 문맥에서 node()를 평가하는 간단한 규칙이 포함되어 있다. 이 함수는 즉시 결과를 반환하며(즉, 비동기적이지 않다), 결과가 노드 문맥에 의존하기 때문에 Table 1의 규칙들과는 분리되어 있다.

<img width="828" height="338" alt="image" src="https://github.com/user-attachments/assets/593a7a51-ea8e-40bf-a624-e6bb431c494a" />
<br>
<br>

종료 및 종료 처리 규칙 또한 시스템 수준에서 평가된다. 종료된 프로세스들은 Table 4에서 볼 수 있듯이 시스템에서 제거된다. Table 4의 마지막 규칙은 새로운 노드를 생성하는 다소 특이한 동작을 다룬다. 이 동작은 의미론 내에서 보기 드문 예외이며, 일반적인 side eff 규칙에 포함시키는 것이 자연스러워 보일 수 있다. 하지만 이 동작의 결과가 전체 노드의 생성이기 때문에 일반적인 패턴에 적합하지 않다. 더욱 일반화된 패턴으로 만들기엔 매력적이지 않았기 때문에, 이 규칙은 별도로 이곳에 명시되어 있다. 이 규칙은 새로운 노드를 생성하며, 새로운 노드 식별자를 갖도록 보장한다.


**노드 수준의 입력 및 출력 규칙** Table 5와 Table 6에는 입력 및 출력 규칙이 담겨 있다.
<img width="896" height="668" alt="image" src="https://github.com/user-attachments/assets/239ed479-0bca-4fa2-b181-ac655c81d9e4" />
<br>
<br>

출력 규칙(Table 5)은 메시지를 시스템 메시지 큐(ether)에 전달하는 반면, internal 규칙은 단순히 출력이 아닌 표현식 평가를 시스템 수준으로 올려주는 역할을 한다(위에서 논의한 바와 같다).

입력 규칙(Table 6)에 대해 주목할 점은, 이 규칙들이 송신자와 수신자 쌍에 대해 임의의 순서로 적용될 수 있다는 것이다. 이는 서로 다른 송신자 및 수신자 간의 메시지가 재정렬될 수 있음을 의미한다. 하지만 동시에 다음과 같은 문제가 발생한다: 어떤 특정한 (송신자, 수신자) 쌍이 영원히 고려되지 않을 수 있다는 것이다. 이는 어떤 메시지의 전달이 영원히 지연될 수도 있다는 뜻이다. 이러한 공정하지 않은(non-fair) 상황에서는 많은 속성들을 증명할 수 없게 된다. 이 문제를 다루기 위해, 우리는 이후 섹션 5에서 공정성(fairness) 규칙을 명시해야 한다.

또한, 어떤 프로세스가 종료되었음에도 여전히 그 프로세스로 메시지를 보내는 것을 막는 규칙이 없기 때문에, 그런 메시지를 결국 제거하기 위한 규칙들(= missing-rules)이 필요하다. $`missing_{node}`$ 규칙은 이와 같은 메시지를 제거하는 역할 외에도, 다음과 같은 노드 컨트롤러 응답 메시지를 보내는 역할도 수행한다: link-reply (noproc), spawn-reply (a useless pid). 이 응답 메시지들은 ncEffect() 함수를 이용해 생성되며, 이 함수는 섹션 4에서 정의된다.

exit-rule은 프로세스가 외부 요인에 의해 비정상적으로 종료되는 경우를 처리한다. 종료 사유는 링크된 프로세스의 종료이거나 명시적인 exit() 호출일 수 있다. 수신 프로세스는 종료되며, 이 사실은 결국 노드 컨트롤러에게 통지된다. 이 메시지 전달 메커니즘은 완전히 비동기적(asynchronous) 이다. 자기 자신에게 보내는 메시지조차도 시스템 메시지 큐를 통해 전달된다. 마지막으로, 노드 컨트롤러에게 보내는 메시지는 Table 7에서 다룬다.



**노드 컨트롤러 (메타)규칙** 
표 7에는 노드 컨트롤러를 위한 메타 규칙들이 포함되어 있다.
이 규칙들은 노드 컨트롤러 신호가 어떻게 처리되는지를 설명한다.

<img width="869" height="247" alt="image" src="https://github.com/user-attachments/assets/adcc43fc-3ca9-4ce4-9ee6-9220c5bb8487" />
<br>
<br>

두 가지 경우가 있다.
일반적인 경우(nc-규칙)에서는 신호가 최종 목적지에 도달했으며, 이 노드 컨트롤러에서 처리되기만 하면 된다.
두 번째 경우(nc_forward-규칙)에서는 원격 pid를 포함하는 신호를 송신 측에서 처리하는 경우이다.
즉, 신호는 로컬 작업을 수행한 뒤 전달되어야 한다.
이러한 작업들과 응답을 명세하는 것이 이 규칙들의 핵심이다.
작업들(즉, 함수 ncEffect의 정의)은 4절에서 제시된다.

로컬 신호와 원격 신호를 구분하기 위해 함수 destNid()가 사용되며, 이는 표 8에 정의되어 있다.

노드 컨트롤러는 별도의 메시지 큐를 갖지 않는다는 점에 유의해야 한다.
노드 컨트롤러에 전송된 신호는 도착 즉시 소비된다.
따라서 노드 컨트롤러에는 선택적 수신(selective receive)이 존재하지 않는다.

nc-규칙과 ncforward-규칙은 전이(transition)에 하나 이상의 동작이 붙을 수 있다는 점에서 특별하다.
즉, 입력 동작과 출력 동작(들)을 모두 가질 수 있다.
하나의 전이에 대해 여러 동작이 존재할 수 있다는 해석은 단순하다.
이러한 동작들은 순차적으로 수행되며, 이는 마치 하나의 동작을 가진 여러 개의 연속적인 전이들이 있는 것과 같다.
함수 mkAction()은 메시지 시퀀스로부터 여러 동작을 생성하는 데 사용된다.
