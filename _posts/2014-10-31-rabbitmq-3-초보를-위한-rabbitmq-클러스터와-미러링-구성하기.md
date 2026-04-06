---
title: "[RabbitMQ] 3. 초보를 위한 RabbitMQ 클러스터와 미러링 구성하기"
date: 2014-10-31 23:31:11 +0900
categories: [IT기타]
---

2장까지 계정설정하고 3장에서 exchange 한다고 예고까지 해놓고 뜬금없이 클러스터와 미러링 구성하기를 작성하게 되었습니다. 방금 회사에서 클러스터랑 미러링을 구성해서 까먹기전에 후딱 써야하는 조금함때문인거 같습니다. exchange 설명 달기도 귀찮기도 하구요. 여튼 시작합니다.

## 0. 목표와 작업 순서

메시징 서비스를 하는데 큐의 의존도가 매우 커졌습니다. 그래서 고가용성 큐를 구성하려고 합니다. 다행히 RabbitMQ에서 완전 짱짱맨인 클러스터, 미러링 기능을 제공하니 그냥 그거 가져다 쓰면 될거 같습니다. 작업 순서는 대충 다음과 같습니다.

1. 서버간 얼랭 쿠키 맞추기
2. 클러스터 구성하기
3. 클러스터 노드 타입 변경
4. 미러링 구성하기
5. 클러스터에서 빠져나오기

본 예제는 3대의 서버를 클러스터, 미러링 하는 것을 기준으로 작성되고 있습니다. 사실 생각보다 무지 쉽습니다. (사이트에서 하라고 하는대로만 잘 따라하면 됩니다. 물론 전 청개구리 모드가 작동하여 하지말라는 순서대로 하다가 MQ 한대 바보로 만들고 두대는 꼬여서 전체 재설치하는 곤욕을 치르기도 했습니다. 으하하!

## 1. 서버간 얼랭 쿠키 맞추기

서버간 얼랭 쿠키를 맞춰줘야 합니다. 자세한 이유는 저도 Document 읽기 귀찮아 생략합니다. 어쨋든 맞춰줘야 한다고 합니다.

1. Erlang 쿠키 위치 확인
   보통 리눅스의 경우 "/var/lib/rabbitmq/.erlang.cookie" 이나 "$HOME/.erlang.cookie" 에 있다고 합니다. 이건 OS나 설치환경 차이가 있으니 각자 알아서 찾는 방법밖에 없습니다. 여튼 클러터를 구성할 3대의 서버중 1대를 기준으로 잡아 scp로 파일 카피를 하거나 vi화면으로 copy&paste 해버리면 됩니다. 혹시 모르니 RabbitMQ를 꺼놓고 작업하도록 합시다.
2. 복사가 제대로 안되어 RabbitMQ가 작동하지 않을때...
   파일 복사가 제대로 이루어지지 않으면, 보통 vi 로 copy&paste하다 삑사리 나면 발생합니다 하하하, RabbitMQ가 구동되지 않습니다. 이럴때는 그냥 쿠키 파일을 지워버리고 다시 시작하면 됩니다. 그럼 새 쿠키카 생성되는데, 여기에 다시 작업하시면 됩니다.

## 2. 클러스터 구성하기

정말 별거 없습니다. 그냥 가이드([https://www.rabbitmq.com/clustering.html](https://www.rabbitmq.com/clustering.html "https://www.rabbitmq.com/clustering.html")) 에서 시키는데로 따라하면 됩니다.

1. 각 서버 상태 확인
   클러스터링이 되기 전 'rabbitmqctl cluster\_status' 명령으로 각 서버 상태를 살펴봅니다. (아래 코드들은 가이드에서 그냥 베껴왔습니다)

   ```bash
   # 1번서버 확인, 자기 자신 노드만 확인됨 
   rabbit1$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit1 ...
   [{nodes,[{disc,}]},{running_nodes,}]
   ...done.
   
   # 2번 서버 확인, 자기 자신 노드만 확인됨
   rabbit2$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit2 ...
   [{nodes,[{disc,}]},{running_nodes,}]
   ...done.
   
   # 3번 서버 확인, 자기 자신 노드만 확인됨
   rabbit3$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit3 ...
   [{nodes,[{disc,}]},{running_nodes,}]
   ...done.
   ```
2. 서버들을 클러스터로 묶기
   작업 시작전 주의사항으로 rabbitmqctl로 노드를 정지시킬때 반드시 stop\_app을 사용해야합니다. 실수로 stop을 해버리면 rabbitmqctl 명령 자체가 먹지 않습니다. 아마 stop은 RabbitMQ node 자체를 종료 시켜버리고 stop\_app은 RabbitMQ의 Management Application을 정지시키기 때문이 아닌가 합니다. 그리고 Cluster를 묶을때 해당 서버의 hostname을 적어 주어야 하는데 정말! 반드시! hostname을 사용해야 합니다. IP이런거 절대 안됩니다. 제가 이거로 반나절 날렸습니다. 어흐흐흐흑! 그리고 /etc/hosts 에 상대 서버의 hostname을 등록할때는 상대 서버에서 cluster\_status로 확인되는 hostname으로 등록해주도록 합니다. hostname을 동일하게 맞추지 않고 구성을 해보지 않아 꼭 맞춰야 되는건지는 알수 없지만 그냥 안전하게 맞추도록 합시다. (그거까지 테스트하기 귀찮아요)이제 1번 서버를 기준으로 클러스터를 묶도록 하겠습니다. 기준이 되는 1번서버는 별다른 작업을 해줄 필요가 없습니다. 그러니 2번 서버부터 작업을 시작해 봅시다.

   ```bash
   #2번서버 작업, 
   #관리 앱 정지
   rabbit2$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit2 ...done.
   
   # 1번 서버로 붙는다. hostname의 경우 1번서버에서 cluster_status로 이름을 확인하여 맞춰 설정하도록 하자.
   rabbit2$ rabbitmqctl join_cluster --ram rabbit@rabbit1
   Clustering node rabbit@rabbit2 with  ...done.
   rabbit2$ rabbitmqctl start_app
   Starting node rabbit@rabbit2 ...done.
   ```

   3번 서버도 마저 작업합니다. 2번 설정할때와 동일합니다.

   ```bash
   rabbit3$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit3 ...done.
   rabbit3$ rabbitmqctl join_cluster rabbit@rabbit2
   Clustering node rabbit@rabbit3 with rabbit@rabbit2 ...done.
   rabbit3$ rabbitmqctl start_app
   Starting node rabbit@rabbit3 ...done.
   ```

   서버별로 cluster\_status 명령을 사용하여 실제 서로 노드를 인식하는지 확인해 봅시다. 클러스터링이 잘 되었다면 status에서 더이상 혼자가 아닌 Node([http://www.youtube.com/watch?v=R85lOwRsnvk](http://www.youtube.com/watch?v=R85lOwRsnvk "http://www.youtube.com/watch?v=R85lOwRsnvk"))를 볼 수 있습니다. 상태를 보면 ram, disc 이런게 좀 신경쓰이는데 이건 곧 설명하도록 하겠습니다.

   ```bash
   #1번 서버 확인
   rabbit1$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit1 ...
   [{nodes,[{disc,},{ram,}]},
    {running_nodes,}]
   ...done.
   
   #2번 서버 확인
   rabbit2$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit2 ...
   [{nodes,[{disc,},{ram,}]},
    {running_nodes,}]
   ...done.
   
   #3번 서버 확인
   rabbit3$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit3 ...
   [{nodes,[{disc,},{ram,}]},
    {running_nodes,}]
   ...done.
   ```
3. 클러스터 최종 확인
   클러스터로 묶인 RabbitMQ들은 서로간의 계정, Virtual Host, policy, Queue, Exchange 등을 모두 공유합니다. 따라서 1번 서버에만 붙어도 2번, 3번 서버의 Queue에 Access할 수 있습니다. 네! 짱짱맨입니다! 간단한 테스트로 Management Plugin으로 접속하여 1번 서버에서 Queue를 만들고 메세지를 Publish해보면 2번, 3번 서버에서도 인식이 되는것을 확인할 수 있습니다.

## 3. 클러스터 노드 타입 변경

rabbitmqctl cluster\_ctl 로 상태를 보면 dics, ram 이란 녀석들이 보입니다. 각 노드들이 현재 상태를 disc로 작동하는지 ram으로 작동하는지를 보여줌니다. 이 두개의 옵션에는 차이가 있는데 ram의 경우 node의 data가 ram에만 상주하여 작동 속도가 빠르고 disc의 경우 node의 data들이 disc에 기록되며 작동한다고 합니다. RabbitMQ 클러스터를 구성할 경우 서버 1대는 반드시 disc로 구성이 되어야하는데, 그 이유는 정말 재수가 없어서 서버가 전체다 내려갔을때 disc로 구성된 녀석을 통해 어느정도 복구가 되도록 하는거 같습니다. 물론 이건 문서를 대충 읽어서 정확한 내용은 아닙니다. 테스트 또한 해보진 않았습니다. 네, 저도 불안하긴 합니다. 여튼 1대는 반드시 disc로 구성하라고 합니다. (뭔가 찜찜하다.)

1. Cluster Node Type 변경
   저는 1번과 3번을 빠른 ram으로 2번 서버를 disc로 구성하려 합니다. 다음과 같이 합니다.

   ```bash
   # disc로 작동할 2번 서버 먼저 작업한다. node중에 disc가 하나라도 없는 상황이 될 수 있으면 변경이 되지 않는다.
   rabbit2$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit2 ...done.
   rabbit2$ rabbitmqctl change_cluster_node_type disc
   Turning rabbit@rabbit2 into a disc node ...
   ...done.
   Starting node rabbit@rabbit2 ...done.
   
   # 1번 서버 작업
   rabbit1$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit1 ...done.
   rabbit1$ rabbitmqctl change_cluster_node_type ram
   Turning rabbit@rabbit1 into a ram node ...
   rabbit1$ rabbitmqctl start_app
   Starting node rabbit@rabbit1 ...done.
   
   # 3번 서버 작업
   rabbit3$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit3 ...done.
   rabbit3$ rabbitmqctl change_cluster_node_type ram
   Turning rabbit@rabbit3 into a ram node ...
   rabbit3$ rabbitmqctl start_app
   Starting node rabbit@rabbit3 ...done.
   ```
2. 확인하기
   1번서버에서 rabbitmqctl cluster\_status로 cluster type이 잘 변경되었는지 확인합니다. 그리고 1번 서버를 stop/start를 하며 2번 서버로 cluster\_status를 체크해보며 node 구성에서 1번 서버의 RabbitMQ가 빠졌다가 다시 들어오는것을 확인합니다. 모든게 잘 작동하면 클러스터게 제대로 구성되었다고 볼 수 있습니다.

## 4. 미러링 구성하기

3번까지 클러스터를 구성하기는 했는데 아직 고가용성이라고 볼수는 없습니다. 클러스터는 서로의 내용을 공유만 할 뿐이어서 불의의 사고로 1번서버가 죽어버린다면 2번, 3번 서버를 통해서 1번서버 큐의 내용을 읽을 수 없게 되며 1번에 남아있던 message들은 모두 증발해 버립니다. 따라서 고가용성을 위해 클러스터의 노드들이 서로의 내용을 복사하여 저장하도록 해주어야 합니다. 노드간 미러링을 구성합니다!

1. 미러링 정책 설정하기
   클러스터만 잘 구성되었다면 미러링은 정말 쉽습니다. 그냥 정책을 정해주는 명령 하나만 날려주면 됩니다. 전 그냥 다 미러링할 계획이라 ha-mode를 all로 했습니다.  옵션별 자세한 내용은 가이드 참조 바랍니다. ([https://www.rabbitmq.com/ha.html](https://www.rabbitmq.com/ha.html "https://www.rabbitmq.com/ha.html"))

   ```bash
   # ha-all은 새로 추가되는 정책의 이름, ^ha\.은 regexp로 표현된 미러링할 queue 이름, 뒤의 json은 정책 세부 사항,
   rabbitmqctl set_policy ha-all "^ha\." '{"ha-mode":"all"}'
   ```
2. Virtual Host 별로 정책 설정해주기
   1번의 내용은 default virtual host에 적용이 됩니다. 새로 만든 virtual host에 ha 정책을 정해주려면 -p 옵션을 사용해야 합니다.

   ```bash
   # myQueue Virtual Host 속한 이름에 .ha 가 포함된 Queue를 모두 미러링 하는 정책
   rabbitmqctl set_policy -p myQueue ha-all "^.*\.ha.*" '{"ha-mode":"all"}'
   ```
3. ha-sync-mode 에 대해
   미러링을 하면 node간 모든 내용들이 복사됩니다. 하지만 고민해야할 것이 하나 있습니다. 클러스터에 새로운 node가 추가되었을때 이 node에 다른 node들이 가지고 있는 과거의 Data를 Sync해야할 것인가? 에 대한 문제입니다. RabbitMQ는 기본적으로 새로 추가된 node(죽었다 살아난 node 포함)에 기존 node의 과거의 data들을 복사하지 않도록 합니다. Data Sync하는 동안 해당 Queue는 무응답 상태가 되어 버려 서비스 가용성에 좋지 않은 영향을 미칠수 있기 때문입니다. 그럼에도 불구하고 추가되는 노드에 반드시 과거 Data가 Sync되어야 한다면 ha-sync-mode 옵션으로 설정할 수 있습니다.

   ```bash
   # ha-sync-mode를 automatic으로 설정, 새로 들어오는 node에도 과거의 data가 모두 복사된다.
   rabbitmqctl set_policy -p myQueue ha-all "^.*\.ha.*" '{"ha-mode":"all", "ha-sync-mode":"automatic"}'
   ```
4. Management Plugin 을 이용한 테스트 및 관리
   사실 미러링은 그냥 Virtual Host에 적용되는 정책에 따라 변할뿐입니다. 그리고 이러한 정책은 Management Plugin에서 Admin=>Policies 메뉴를 이용하면 좀더 편리하게 설정을 할 수 있습니다.

## 5. 클러스터에서 빠져나오기

클러스터를 구성했으면 클러스터에서 빠져 나올수도 있어야 합니다.

1. 클러스터에서 빠져나오기
   이것도 간단합니다. 클러스터에서 뺄 서버에 가서 명령 하나만 쳐주면 됩니다. 3번 서버를 제거해보도록 하겠습니다.

   ```bash
   # 3번 서버를 클러스터에서 제거
   rabbit3$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit3 ...done.
   rabbit3$ rabbitmqctl reset
   Resetting node rabbit@rabbit3 ...done.
   rabbit3$ rabbitmqctl start_app
   Starting node rabbit@rabbit3 ...done.
   
   # 2번 서버에서 확인해보기
   rabbit2$ rabbitmqctl cluster_status
   Cluster status of node rabbit@rabbit2 ...
   [{nodes,[{disc,}]},
    {running_nodes,}]
   ```
2. 혹 클러스터에서 제거해야할 node가 정상적으로 작동이 되지 않는 상태라면?
   다른 서버에서 원격으로 제거할 수 있습니다.

   ```bash
   # 3번 서버가 정지되어있다면...
   rabbit3$ rabbitmqctl stop_app
   Stopping node rabbit@rabbit3 ...done.
   
   # 1번이나 2번 서버에서 원격으로 제거 한다. 나중에 3번 서버는 reset하여 클러스터에서 완전히 빠지도록 해준다.
   rabbit2$ rabbitmqctl forget_cluster_node rabbit@rabbit3
   Removing node rabbit@rabbit1 from cluster ...
   ...done.
   ```

마지막으로 클러스터와 미러링 구성이 잘 되었나 확인하는것도 역시 Management Plugin을 사용하면 정말 편합니다. 관리자 계정으로 접속하면 home 화면에 cluster가 어떻게 구성되었는지 뙇! 보여줍니다. 미러링이 잘되었는지는 Queue 메뉴에 들어가면 확인 가능합니다. Queue 리스트를 보면 미러링 되는 큐는 이름 옆에 (+2) 이런 표식이 나옵니다. 미러링 되고 있는 node의 수를 보여주는 값입니다. 저게 생기지 않는다면 클러스터에서 공유만 되고 미러링이 되고 있지 않는 Queue 입니다.
오늘건 좀 길었네요. 헉헉.
