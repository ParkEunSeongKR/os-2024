# Week 2 Homework

1.다음 플래그와 함께 process-run.py를 실행하세요: -l 5:100,5:100. CPU 활용률(예: CPU가 사용 중인 시간의 백분율)은 어떻게 되어야 할까요? 왜 그렇게 알 수 있나요? -c와 -p 플래그를 사용해 여러분의 생각이 맞는지 확인해 보세요.

예상 CPU 활용률은 100%일거다.

코드 결과
```
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1
  2        RUN:cpu         READY             1
  3        RUN:cpu         READY             1
  4        RUN:cpu         READY             1
  5        RUN:cpu         READY             1
  6           DONE       RUN:cpu             1
  7           DONE       RUN:cpu             1
  8           DONE       RUN:cpu             1
  9           DONE       RUN:cpu             1
 10           DONE       RUN:cpu             1

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

2.이제 다음 플래그로 실행해 보세요: ./process-run.py -l 4:100,1:0. 이 플래그는 4개의 명령어(모두 CPU 사용)를 가진 하나의 프로세스와 I/O를 실행하고 완료될 때까지 기다리는 하나의 프로세스를 지정합니다. 두 프로세스를 모두 완료하는 데 얼마나 걸리나요? -c와 -p를 사용해 여러분의 생각이 맞는지 확인해 보세요.

첫 번째 프로세스는 CPU 작업 4개를 수행하고, 두 번째 프로세스는 I/O 작업 1개를 수행합니다. 프로세스 전체가 완료되는 데 걸리는 총 시간은 6틱입니다.

코드결과
```
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1
  2        RUN:cpu         READY             1
  3        RUN:cpu         READY             1
  4        RUN:cpu         READY             1
  5           DONE        RUN:io             1
  6           DONE       BLOCKED                           1
  7           DONE       BLOCKED                           1
  8           DONE       BLOCKED                           1
  9           DONE       BLOCKED                           1
 10           DONE       BLOCKED                           1
 11*          DONE   RUN:io_done             1

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

3.프로세스의 순서를 바꿔 보세요: -l 1:0,4:100. 이제 어떻게 되나요? 순서를 바꾸는 것이 중요한가요? 왜 그런가요? (항상 그렇듯이 -c와 -p를 사용해 여러분의 생각이 맞는지 확인해 보세요)

-> 첫 번째 프로세스는 I/O를 수행하고, 두 번째 프로세스는 CPU 작업 4개를 수행합니다. 이전과 달리 먼저 I/O를 수행하는 첫 번째 프로세스 때문에 전체 실행 시간이 약간 늘어날 수 있습니다


4.이제 다른 플래그들을 살펴보겠습니다. 중요한 플래그 중 하나는 -S인데, 이는 프로세스가 I/O를 실행할 때 시스템이 어떻게 반응하는지 결정합니다. 플래그를 SWITCH_ON_END로 설정하면 한 프로세스가 I/O를 수행하는 동안 시스템은 다른 프로세스로 전환하지 않고 해당 프로세스가 완전히 끝날 때까지 기다립니다. 다음 두 프로세스를 실행할 때 어떤 일이 일어나나요(-l 1:0,4:100 -c -S SWITCH_ON_END), 하나는 I/O를 수행하고 다른 하나는 CPU 작업을 수행합니다?

-> 프로세스가 I/O를 발행할 때 다른 프로세스로 전환하지 않고, I/O를 발행한 프로세스가 완전히 끝날 때까지 기다립니다. 첫 번째 프로세스가 I/O를 수행하고, 두 번째 프로세스는 CPU 작업을 수행하며 기다립니다. 따라서 첫 번째 프로세스가 완료될 때까지 시스템은 다른 프로세스를 실행하지 않습니다.

5.이제 같은 프로세스를 실행하되, 한 프로세스가 I/O를 기다리는(WAITING) 동안 다른 프로세스로 전환되도록 switching 동작을 설정해 봅시다(-l 1:0,4:100 -c -S SWITCH_ON_IO). 이제 어떤 일이 일어나나요? -c와 -p를 사용해 여러분의 생각이 맞는지 확인해 보세요.

-> 프로세스가 I/O를 발행할 때 WAITING 상태로 전환되고, 다른 프로세스로 전환됩니다. 첫 번째 프로세스가 I/O를 발행하면 기다리는 동안 두 번째 프로세스가 실행될 수 있습니다. 이는 시스템 자원을 더 효율적으로 사용할 수 있게 합니다.

6.또 다른 중요한 동작은 I/O가 완료될 때 수행할 작업입니다. -I IO_RUN_LATER를 사용하면 I/O가 완료될 때 그것을 실행한 프로세스가 반드시 즉시 실행되는 것은 아닙니다. 오히려 그 시점에 실행 중이던 프로세스가 계속 실행됩니다. 이러한 프로세스 조합을 실행하면 어떤 일이 일어나나요? (./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p) 시스템 자원이 효율적으로 활용되고 있나요?

-> 첫 번째 프로세스는 I/O를 발행하지 않고, 나머지 세 프로세스는 CPU 작업을 실행합니다. IO_RUN_LATER 설정은 I/O가 완료될 때 해당 프로세스가 즉시 실행되는 것이 아니라, 시스템에 의해 결정됩니다. SWITCH_ON_IO는 I/O를 기다리는 동안 다른 프로세스로 전환합니다. 따라서 시스템 자원이 더 효율적으로 사용될 수 있습니다.

7.이제 같은 프로세스를 실행하되, -I IO_RUN_IMMEDIATE를 설정해 I/O를 실행한 프로세스를 즉시 실행하도록 해 봅시다. 이 동작은 어떻게 다른가요? 방금 I/O를 완료한 프로세스를 다시 실행하는 것이 왜 좋은 생각일 수 있을까요?

-> I/O를 발행한 프로세스가 완료되면 즉시 실행됩니다. 따라서 I/O가 자주 발생하는 프로세스가 빠르게 다시 실행될 수 있어 시스템 반응성을 향상시킬 수 있습니다. 이는 특히 I/O 집중적인 작업에서 유용할 수 있습니다.

8.이제 무작위로 생성된 프로세스로 실행해 봅시다: -s 1 -l 3:50,3:50 또는 -s 2 -l 3:50,3:50 또는 -s 3 -l 3:50,3:50. 추적 결과가 어떻게 될지 예측해 보세요. -I IO_RUN_IMMEDIATE 플래그와 -I IO_RUN_LATER 플래그를 사용할 때 어떤 일이 일어나나요? -S SWITCH_ON_IO와 -S SWITCH_ON_END를 사용할 때 어떤 일이 일어나나요?

-> 두 프로세스가 각각 CPU 작업과 I/O 작업을 번갈아가며 실행합니다. 각 설정에 따라 CPU 및 IO 작업의 스케줄링이 달라지며, 이에 따라 전체 실행 시간 및 시스템의 CPU 및 IO 활용률이 달라질 수 있습니다.

