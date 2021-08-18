# 6.4 Hardware Support for Synchronization

# Hardware Support

- 소프트웨어뿐만이 아닌 하드웨어 수준에서의 명령어 에서도 synchronization 지원이 필요해서
    - 어셈블리어에서의 스케줄링이 가능 할려면 하드웨어 수준의  synchronization이 필요한듯
- syncrhonization 툴 자체로도 사용 가능
- 더 추상적인 메카니즘의 구현에 사용 가능
    - 그래서 하드웨어 수준에서 구현된 명령어(instruction)을 고수준 언어인 JAVA에서 사용하기 위해 별도의 API가 있다고 생각됩니다.

    ## 어셈블리어 실행과정 예시

    ![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled.png)

작업목표: A의 8번째 인덱스의 값에 h를 더하고 A의 12번째 인덱스에 넣는다.

LDUR(load) - X22레지스터(A[0])에서 8만큼 이동하고 X9레지스터에 저장한다.

ADD - X21레지스터와 X9레지스터를 합하고 X9레지스터에 저장한다.

STUR(store) - X22레지스터(A[0])에서 12만큼 이동하고 X9를 저장한다

> 위의 어셈블리어 작업 수행중에 interrupt가 발생할 수 도 있다는 전제하에 하드웨어 수준의 동기화 툴이 필요하다고 하는것 같습니다.

### 주요 방법들

- Memory Barriers of fencers - 강의에선 제외되었지만 원본 ppt에 설명이 있음
- Hardware Instructions
- Atomic Variables

# Atomicity

물질 구조의 최소단위인 원자처럼 

작업의 최소 단위로서 더이상 interrupt 될수 없는 수준을 말한다.

현대 컴퓨터 시스템은 자체적으로 atomic한 Hardware Instruction을 제공한다.

# Hardware Instruction

어셈블리어 혹은 회로 수준으로 구현된 기능을 코드로 설명해주고 있다.

# Test_and_Set

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%201.png)

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%202.png)

# Compare_and_swap

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%203.png)

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%204.png)

# Atomic Variable

compare and swap 명령어는 atomic variable을 만들때 자주 사용된다.

단일 변수에 대해 race condition이 발생하면 mutual exclusion할 수 있는 atomic operation을 만들 수 있다.

# Peterson's Solution(JAVA)

producer와 consumer 두 스레드가 동시에 실행될때 JAVA의 API를 통해 해결하는 방법을 설명

```java
public class Peterson1 {
	static int count = 0;
	static int turn = 0;
	static boolean[] flag = new boolean[2];

	public static void main(String[] args) throws Exception {
		Thread t1 = new Thread(new Producer());
		Thread t2 = new Thread(new Consumer());
		t1.start(); t2.start();
		t1.join(); t2.join();
		System.out.println(Peterson1.count);
	}
}
```

Producer

```java
static class Producer implements Runnable {
	@Override
	public void run() {
		for (int k = 0; k < 10000; k++) {
			/* entry section */
			flag[0] = true;
			turn = 1;
			while (flag[1] && turn == 1)
			;
			/* critical section */
			count++;
			/* exit section */
			flag[0] = false;
			/* remainder section *
		}
	}
}
```

Consumer

```java
static class Consumer implements Runnable {
	@Override
	public void run() {
		for (int k = 0; k < 10000; k++) {
		/* entry section */
		flag[1] = true;
		turn = 0;
		while (flag[0] && turn == 0)
		;
		/* critical section */
		count--;
		/* exit section */
		flag[1] = false;
		/* remainder section */
		}
	}
}
```

JAVA에서는 Peterson's Solution을 그대로 옮겨서 구현하면 race condition을 고려하지 않고 구현하게 된다. 멀티 스레드 환경을 가정하고 구현할려면 Atomic Type을 사용해서 구현해야 한다.

JAVA의 AtomicBoolean은 하드웨어 기반의 Compare_and_Swap를 사용해서 한번에 하나의 스레드만 변수의 값을 변경하도록 하고 있다. 

AtomicBoolean은 일반 Boolean과 같이 True/False 만 가질 수 있다.

제공되는 메서드는 다음과 같다.

- get(): 변수값 조회
- set(): 변수값 변경
- getAndSet(): 변수값 조회 후 변경
- compareAndSet(): 변수값 비교 후 변경

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%205.png)

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%206.png)

![Untitled](6%204%20Hardware%20Support%20for%20Synchronization%20f8215bb7bcc449729ae789359e21b842/Untitled%207.png)