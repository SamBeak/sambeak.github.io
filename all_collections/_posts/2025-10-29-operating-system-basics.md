---
layout: post
title: 💻 운영체제(OS) 기본 원리 이해하기
date: 2025-10-29
categories: ["SamBeak", "OS", "Operating System", "Computer Science", "운영체제"]
---

# 운영체제란 무엇인가

<br>
컴퓨터를 켜면 가장 먼저 만나는 것이 운영체제다. <br>
Windows, macOS, Linux 등 우리가 매일 사용하는 이 프로그램들은 <br>
단순히 예쁜 화면을 보여주는 것 이상의 역할을 한다. <br><br>

운영체제를 비유하자면 건물의 관리인과도 같다. <br>
건물에 입주한 여러 회사(응용 프로그램)들이 <br>
전기(CPU), 물(메모리), 공간(저장장치) 등의 자원을 <br>
효율적으로 나눠 쓸 수 있도록 중재하고 관리하는 역할을 한다. <br><br>

처음 프로그래밍을 배울 때는 운영체제의 존재를 크게 의식하지 않는다. <br>
그저 코드를 작성하고 실행하면 결과가 나온다. <br>
하지만 조금 더 깊이 들어가다 보면 <br>
우리가 작성한 프로그램이 실제로 어떻게 실행되는지, <br>
메모리는 어떻게 관리되는지, 파일은 어떻게 저장되는지 <br>
궁금해지는 순간이 온다. <br><br>

> ## 왜 운영체제를 이해해야 할까?

<br>
개발자라면 운영체제의 기본 원리를 이해하는 것이 중요하다. <br><br>

첫째, **성능 최적화**를 할 수 있다. <br>
프로세스와 스레드의 차이를 알면 멀티태스킹을 효율적으로 구현할 수 있고, <br>
메모리 관리 원리를 알면 메모리 누수를 방지할 수 있다. <br><br>

둘째, **문제 해결 능력**이 향상된다. <br>
프로그램이 느려지거나 멈추는 이유를 파악할 수 있고, <br>
데드락이나 경쟁 상태 같은 문제를 예방할 수 있다. <br><br>

셋째, **시스템 레벨 이해**가 깊어진다. <br>
네트워크, 데이터베이스, 서버 등 <br>
모든 컴퓨팅 시스템의 기반이 운영체제이기 때문이다. <br><br>

# 운영체제의 주요 역할

<br>
운영체제는 크게 4가지 핵심 역할을 수행한다. <br><br>

## 🟢 프로세스 관리 (Process Management)

<br>
프로세스는 실행 중인 프로그램을 의미한다. <br>
운영체제는 여러 프로세스가 CPU를 효율적으로 나눠 쓸 수 있도록 <br>
스케줄링(Scheduling)을 수행한다. <br><br>

예를 들어, 음악을 들으면서 문서 작성을 하고 <br>
동시에 파일을 다운로드받을 수 있는 이유는 <br>
운영체제가 각 프로세스에 CPU 시간을 번갈아가며 할당하기 때문이다. <br><br>

**프로세스의 생명 주기:**

1. **생성(New)**: 프로세스가 만들어지는 단계 <br>
2. **준비(Ready)**: CPU를 할당받기를 기다리는 단계 <br>
3. **실행(Running)**: CPU에서 명령을 실행하는 단계 <br>
4. **대기(Waiting)**: I/O 작업이나 이벤트를 기다리는 단계 <br>
5. **종료(Terminated)**: 실행이 완료된 단계 <br><br>

## 🟢 메모리 관리 (Memory Management)

<br>
메모리는 컴퓨터의 작업 공간이다. <br>
운영체제는 한정된 메모리를 여러 프로세스가 <br>
효율적으로 사용할 수 있도록 관리한다. <br><br>

**주요 메모리 관리 기법:**

| 기법 | 설명 |
|------|------|
| 스왑(Swap) | 메모리가 부족할 때 디스크를 보조 메모리로 사용 |
| 페이징(Paging) | 메모리를 고정 크기 블록으로 나누어 관리 |
| 세그먼테이션(Segmentation) | 논리적 단위로 메모리를 분할 |
| 가상 메모리(Virtual Memory) | 물리 메모리보다 큰 주소 공간 제공 |

<br>
가상 메모리는 특히 중요한 개념이다. <br>
예를 들어, 8GB RAM을 가진 컴퓨터에서 <br>
10GB 크기의 프로그램을 실행할 수 있는 이유는 <br>
가상 메모리 덕분이다. <br><br>

운영체제는 현재 사용하지 않는 메모리 부분을 <br>
디스크에 임시 저장(스왑 아웃)하고, <br>
필요할 때 다시 메모리로 가져온다(스왑 인). <br><br>

## 🟢 파일 시스템 관리 (File System Management)

<br>
파일 시스템은 데이터를 저장하고 관리하는 체계다. <br>
운영체제는 파일의 생성, 읽기, 쓰기, 삭제 등을 관리하고 <br>
파일에 대한 접근 권한을 제어한다. <br><br>

**주요 파일 시스템:**

- **NTFS** (Windows): 권한 관리, 암호화, 대용량 파일 지원 <br>
- **APFS** (macOS): SSD 최적화, 스냅샷 기능 <br>
- **ext4** (Linux): 저널링, 대용량 파일 시스템 <br>
- **FAT32**: 호환성이 높지만 4GB 파일 크기 제한 <br><br>

파일 시스템은 단순히 파일을 저장하는 것이 아니라 <br>
메타데이터(파일 이름, 크기, 권한, 수정 날짜 등)도 함께 관리한다. <br><br>

## 🟢 입출력 관리 (I/O Management)

<br>
키보드, 마우스, 모니터, 프린터, 하드디스크 등 <br>
다양한 하드웨어 장치와의 통신을 관리한다. <br><br>

운영체제는 각 장치마다 다른 특성을 추상화하여 <br>
응용 프로그램이 일관된 방식으로 장치를 사용할 수 있게 한다. <br>
이를 위해 **디바이스 드라이버(Device Driver)**를 사용한다. <br><br>

# 프로세스와 스레드의 차이

<br>
프로세스와 스레드는 헷갈리기 쉬운 개념이다. <br><br>

> ## 프로세스 (Process)

<br>
프로세스는 **독립적인 실행 단위**다. <br>
각 프로세스는 자신만의 메모리 공간을 갖고 있어 <br>
다른 프로세스의 메모리에 접근할 수 없다. <br><br>

예를 들어, Chrome 브라우저와 VSCode는 <br>
각각 별도의 프로세스로 실행된다. <br>
하나가 충돌해도 다른 것에는 영향을 주지 않는다. <br><br>

**프로세스의 특징:**
- 독립된 메모리 공간 (Code, Data, Stack, Heap) <br>
- 프로세스 간 통신(IPC)이 필요 <br>
- 생성과 전환 비용이 크다 <br><br>

> ## 스레드 (Thread)

<br>
스레드는 **프로세스 내의 실행 단위**다. <br>
같은 프로세스 내의 스레드들은 메모리를 공유한다. <br><br>

예를 들어, 웹 브라우저 한 개(프로세스) 안에서 <br>
여러 탭(스레드)이 동시에 실행되는 것이다. <br>
한 탭에서 영상을 다운로드하는 동안 <br>
다른 탭에서 웹서핑을 할 수 있다. <br><br>

**스레드의 특징:**
- 같은 프로세스 내에서 메모리 공유 <br>
- 빠른 생성과 전환 <br>
- 동기화 문제 발생 가능 (경쟁 상태, 데드락) <br><br>

**비교표:**

| 특성 | 프로세스 | 스레드 |
|------|----------|--------|
| 메모리 | 독립적 | 공유 |
| 생성 비용 | 높음 | 낮음 |
| 통신 | IPC 필요 | 직접 통신 |
| 안정성 | 높음 | 낮음 |

<br>

# CPU 스케줄링 알고리즘

<br>
CPU 스케줄링은 여러 프로세스 중 <br>
어떤 프로세스에 CPU를 할당할지 결정하는 것이다. <br><br>

## 🏷️ FCFS (First-Come, First-Served)

<br>
먼저 도착한 프로세스를 먼저 실행한다. <br>
은행 창구에서 줄을 서는 것과 같다. <br><br>

**장점**: 구현이 간단하고 공평하다 <br>
**단점**: 긴 작업이 먼저 오면 짧은 작업들이 오래 기다린다 (Convoy Effect) <br><br>

## 🏷️ SJF (Shortest Job First)

<br>
실행 시간이 가장 짧은 프로세스를 먼저 실행한다. <br>
평균 대기 시간을 최소화할 수 있다. <br><br>

**장점**: 평균 대기 시간이 짧다 <br>
**단점**: 실행 시간을 미리 알기 어렵고, 긴 작업은 계속 밀릴 수 있다 (Starvation) <br><br>

## 🏷️ RR (Round Robin)

<br>
각 프로세스에 동일한 시간(Time Quantum)을 할당하고 <br>
순환하며 실행한다. <br><br>

**장점**: 공평하고 응답 시간이 빠르다 <br>
**단점**: 문맥 교환(Context Switch) 오버헤드가 발생한다 <br><br>

## 🏷️ Priority Scheduling

<br>
각 프로세스에 우선순위를 부여하고 <br>
우선순위가 높은 프로세스를 먼저 실행한다. <br><br>

**장점**: 중요한 작업을 먼저 처리할 수 있다 <br>
**단점**: 낮은 우선순위 프로세스가 기아 상태(Starvation)에 빠질 수 있다 <br><br>

# 동기화 문제와 해결 방법

<br>
여러 스레드나 프로세스가 공유 자원에 접근할 때 <br>
동기화 문제가 발생할 수 있다. <br><br>

> ## 경쟁 상태 (Race Condition)

<br>
두 개 이상의 프로세스가 동시에 공유 데이터에 접근하여 <br>
결과가 실행 순서에 따라 달라지는 상황이다. <br><br>

**예시:**
```python
# 두 스레드가 동시에 count를 증가시킨다
count = 0

def increment():
    global count
    temp = count
    temp = temp + 1
    count = temp

# Thread 1과 Thread 2가 동시 실행
# 기대: count = 2
# 실제: count = 1 (경쟁 상태 발생 가능)
```

<br>

> ## 임계 영역 (Critical Section)

<br>
여러 프로세스가 공유하는 데이터를 접근하는 코드 영역이다. <br>
임계 영역에는 한 번에 하나의 프로세스만 들어갈 수 있어야 한다. <br><br>

**임계 영역 문제의 해결 조건:**

1. **상호 배제(Mutual Exclusion)**: 한 번에 하나만 실행 <br>
2. **진행(Progress)**: 임계 영역에 아무도 없으면 진입 허용 <br>
3. **유한 대기(Bounded Waiting)**: 기다리는 시간이 유한해야 함 <br><br>

> ## 해결 방법

<br>

### 🔒 뮤텍스 (Mutex)

<br>
뮤텍스는 **상호 배제(Mutual Exclusion)**의 줄임말로, <br>
임계 영역에 하나의 스레드만 접근할 수 있도록 하는 잠금 장치다. <br><br>

```python
import threading

lock = threading.Lock()
count = 0

def increment():
    global count
    lock.acquire()  # 잠금 획득
    try:
        count += 1
    finally:
        lock.release()  # 잠금 해제
```

<br>

### 🔒 세마포어 (Semaphore)

<br>
세마포어는 여러 개의 스레드가 접근할 수 있는 자원의 개수를 제한한다. <br>
주차장의 가용 공간과 비슷한 개념이다. <br><br>

```python
import threading

semaphore = threading.Semaphore(3)  # 최대 3개의 스레드 허용

def access_resource():
    semaphore.acquire()
    try:
        # 공유 자원 사용
        print("자원 사용 중")
    finally:
        semaphore.release()
```

<br>

### 🔒 모니터 (Monitor)

<br>
모니터는 뮤텍스와 조건 변수를 결합한 상위 수준의 동기화 도구다. <br>
자바의 `synchronized` 키워드가 대표적인 예다. <br><br>

# 데드락 (Deadlock)

<br>
데드락은 두 개 이상의 프로세스가 <br>
서로가 가진 자원을 기다리며 무한정 대기하는 상태를 말한다. <br><br>

**일상 생활의 비유:**
A와 B가 좁은 다리에서 마주쳤는데, <br>
A는 B가 비켜주기를 기다리고 <br>
B는 A가 비켜주기를 기다리는 상황이다. <br><br>

> ## 데드락 발생 조건 (4가지 모두 충족)

<br>

1. **상호 배제(Mutual Exclusion)**: 자원은 한 번에 하나의 프로세스만 사용 <br>
2. **점유와 대기(Hold and Wait)**: 자원을 가진 상태에서 다른 자원을 기다림 <br>
3. **비선점(No Preemption)**: 자원을 강제로 빼앗을 수 없음 <br>
4. **순환 대기(Circular Wait)**: 프로세스들이 순환 형태로 자원을 기다림 <br><br>

> ## 데드락 해결 방법

<br>

### 🏷️ 예방(Prevention)
4가지 조건 중 하나를 불가능하게 만든다. <br>
예: 모든 자원을 한 번에 요청하도록 강제 <br><br>

### 🏷️ 회피(Avoidance)
자원 할당 시 데드락 가능성을 미리 검사한다. <br>
예: 은행원 알고리즘(Banker's Algorithm) <br><br>

### 🏷️ 탐지 및 회복(Detection & Recovery)
데드락 발생을 허용하되, 탐지 후 회복한다. <br>
예: 프로세스를 강제 종료하거나 자원을 선점 <br><br>

### 🏷️ 무시
데드락 발생 확률이 낮으면 무시한다. <br>
대부분의 운영체제가 채택하는 방식이다. <br><br>

# 메모리 관리 심화

<br>

> ## 페이지 교체 알고리즘

<br>
가상 메모리 시스템에서 메모리가 부족할 때 <br>
어떤 페이지를 디스크로 내보낼지 결정하는 알고리즘이다. <br><br>

### 🏷️ FIFO (First In First Out)

가장 먼저 들어온 페이지를 교체한다. <br>
구현은 간단하지만 성능이 좋지 않다. <br><br>

### 🏷️ LRU (Least Recently Used)

가장 오랫동안 사용되지 않은 페이지를 교체한다. <br>
실제로 가장 많이 사용되는 알고리즘이다. <br><br>

### 🏷️ LFU (Least Frequently Used)

사용 빈도가 가장 낮은 페이지를 교체한다. <br><br>

### 🏷️ Optimal

미래에 가장 오랫동안 사용되지 않을 페이지를 교체한다. <br>
이론적으로 가장 좋지만 미래를 알 수 없어 구현 불가능하다. <br><br>

> ## 페이지 폴트 (Page Fault)

<br>
프로세스가 접근하려는 페이지가 메모리에 없을 때 발생한다. <br><br>

**처리 과정:**

1. CPU가 페이지 접근 시도 <br>
2. 페이지가 메모리에 없음을 감지 <br>
3. 운영체제에 페이지 폴트 인터럽트 발생 <br>
4. 디스크에서 해당 페이지를 찾음 <br>
5. 빈 프레임에 페이지를 로드 (필요시 페이지 교체) <br>
6. 페이지 테이블 업데이트 <br>
7. 프로세스 재개 <br><br>

페이지 폴트가 자주 발생하면 **스래싱(Thrashing)**이 발생하여 <br>
시스템 성능이 급격히 저하된다. <br><br>

# 실전 예시: Python에서의 멀티스레딩

<br>
운영체제 개념을 실제로 활용하는 예시를 살펴보자. <br><br>

```python
import threading
import time

# 공유 자원
counter = 0
lock = threading.Lock()

def increment_with_lock(name):
    """잠금을 사용한 안전한 증가"""
    global counter
    for _ in range(100000):
        lock.acquire()
        try:
            counter += 1
        finally:
            lock.release()
    print(f"{name} 완료")

def increment_without_lock(name):
    """잠금을 사용하지 않은 위험한 증가"""
    global counter
    for _ in range(100000):
        counter += 1
    print(f"{name} 완료")

# 잠금 사용 예제
print("=== 잠금 사용 ===")
counter = 0
threads = []
for i in range(5):
    t = threading.Thread(target=increment_with_lock, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"최종 counter 값: {counter}")  # 500000 (정확함)

# 잠금 미사용 예제
print("\n=== 잠금 미사용 ===")
counter = 0
threads = []
for i in range(5):
    t = threading.Thread(target=increment_without_lock, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"최종 counter 값: {counter}")  # 500000보다 작을 수 있음 (경쟁 상태)
```

<br>
이 예제는 동기화의 중요성을 보여준다. <br>
잠금을 사용하지 않으면 경쟁 상태로 인해 <br>
예상과 다른 결과가 나올 수 있다. <br><br>

# 운영체제 성능 모니터링

<br>
실무에서 운영체제 성능을 모니터링하는 것은 중요하다. <br><br>

## 🏷️ 주요 성능 지표

<br>

| 지표 | 설명 | 확인 방법 |
|------|------|----------|
| CPU 사용률 | CPU가 얼마나 바쁜지 | top, htop (Linux), 작업 관리자 (Windows) |
| 메모리 사용률 | RAM 사용 정도 | free, vmstat (Linux), 작업 관리자 |
| 디스크 I/O | 디스크 읽기/쓰기 속도 | iostat, iotop (Linux) |
| 네트워크 대역폭 | 네트워크 사용량 | iftop, nethogs (Linux) |
| 컨텍스트 스위치 | 프로세스 전환 빈도 | vmstat (Linux) |

<br>

**Linux 명령어 예시:**

```bash
# CPU와 메모리 사용률 확인
top

# 메모리 상세 정보
free -h

# 디스크 I/O 통계
iostat -x 1

# 프로세스별 메모리 사용량
ps aux --sort=-%mem | head -10

# 시스템 전체 성능 요약
vmstat 1 10
```

<br>

# 마무리

<br>
운영체제는 컴퓨터 시스템의 핵심이다. <br>
프로세스 관리, 메모리 관리, 파일 시스템, I/O 관리 등 <br>
모든 컴퓨팅 자원을 효율적으로 관리한다. <br><br>

개발자로서 운영체제의 기본 원리를 이해하면 <br>
더 효율적인 프로그램을 작성할 수 있고, <br>
성능 문제를 빠르게 진단하고 해결할 수 있다. <br><br>

처음엔 복잡해 보이지만, <br>
실제로 코드를 작성하고 문제를 해결하다 보면 <br>
운영체제의 개념들이 자연스럽게 이해될 것이다. <br><br>

이 글이 운영체제를 이해하는 시작점이 되길 바란다. <br><br>
