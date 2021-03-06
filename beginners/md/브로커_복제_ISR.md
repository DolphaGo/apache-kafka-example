# 브로커, 복제, ISR(In-Sync-Replication)

- 카프카 운영에 있어서 아주 종요함
- Replication (복제)
  - 카프카 아키텍쳐의 핵심
  - 클러스터에 서버에 장애가 생겼을 때 가용성을 보장하는 가장 좋은 방법


- 카프카 브로커 : 카프카가 설치되어 있는 서버 단위를 말함
  - 보통 3개 이상의 브로커를 설치하는 것을 권장함

![](/images/2022-06-02-15-45-58.png)

- 레플리케이션은 파티션의 복제를 의미합니다.
- 레플리케이션이 1이라면, 파티션은 1개만 존재한다는 것이고, 레플리케이션이 2라면 원본 1개와 복제본 1개로 총 2개가 존재합니다.
- 레플리케이션이 3이라면, 원본 1개와 복제본 2개로 총 3개가 됩니다.

![](/images/2022-06-02-15-47-00.png)

- 다만, **브로커 개수에 따라서 replication 개수가 제한이 된다.**
  - 브로커 개수가 3이면, 레플리케이션이 4가 될 수 없다는 것입니다.

- 여기서 원본 파티션을 리더 파티션이라고 부르고, 나머지 복제 파티션을 팔로워 파티션이라고 합니다.
- 리더 + 팔로워 = ISR (In-Sync Replication)

왜 Replication을 사용하는걸까?
- 레플리케이션은 **파티션의 고가용성을 위해 사용**합니다.
- 만약 브로커가 3개인 카프카에서 레플리케이션이 1이고, 파티션이 1인 토픽이 존재한다고 가정해봅시다.
- 갑자기 브로커가 어떠한 이유로 사용 불가하게 된다면, 더이상 해당 파티션은 복구할 수 없습니다.
- 만약 레플리케이션이 2라면 어떻게 되나요?
  - 리더가 죽더라도, 팔로워 파티션이 있기 때문에 복구 가능합니다.
  - 리더가 죽으면 팔로워 파티션이 리더 파티션으로 승계할 수 있기 때문입니다.

리더 파티션과 팔로워 파티션의 역할은 무엇일까요?

- 프로듀서가 토픽의 파티션에 데이터를 전달할 때, **전달받는 주체가 바로 리더 파티션**입니다.
- ack를 통해 고가용성을 유지할 수 있는데
  - ack는 0, 1, all 옵션 3개중 1개를 골라 사용할 수 있다.
  - 0 : 프로듀서는 리더 파티션에 데이터를 전송하고 응답값을 받지 않습니다.
    - 그렇기 때문에 데이터가 제대로 전송됐는지, 나머지 팔로워 파티션에 데이터가 제대로 복제됐는지 알 수 없다.
    - 속도는 빠르지만, 데이터 유실 가능성이 있다.
  - 1 : 리더 파티션에 데이터를 전송하고, 리더 파티션이 데이터를 정상적으로 받았는지 응답값을 받습니다.
    - 다만, 다른 파티션에 제대로 복제가 됐는지 알 수는 없습니다.
    - 즉, 리더 파티션이 데이터를 받은 즉시 장애가 나면, 다른 파티션에서 복제가 되지 않은 상황이라서 데이터 유실 가능성이 있습니다.
  - all: 리더 파티션, 팔로워 파티션에 잘 복제가 되었는지 응답값을 모두 받는다.
    - 나머지 팔로워 파티션에도 제대로 복제가 됐는지 확인하는 작업을 거친다.
    - 데이터 유실 작업은 없다고 보면 되지만, 각각 응답을 받기 때문에 속도가 현저히 느리다는 단점이 있습니다.


레플리케이션이 고가용성을 위해 중요한 역할을 한다면, 레플리케이션이 많을수록 좋은거 아닌가? 할 수 있다.
- 레플리케이션의 개수가 많아지면 브로커의 리소스 사용량도 많아지게 된다.

![](/images/2022-06-02-15-52-36.png)

- 따라서 카프카에 들어오는 데이터량과 retention date, 즉 저장시간을 잘 생각하여 레플리케이션을 지정하는 것이 좋다.
- 3개 이상의 브로커를 사용할 때 레플리케이션은 3으로 설정하는 것을 추천한다.