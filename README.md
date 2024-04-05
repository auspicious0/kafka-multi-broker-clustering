1. 목적: 카프카의 성능 향상을 위해 분산 처리 기능을 추가하고자 한다.

   분산 처리를 수행하는 가장 단순한 방법은 서버를 늘리는 것이다.

   하지만 서버를 늘리게 되면 zookeeper가 1:1로 대응 되어야 하지만, 현재 제품의 용량 limit이 넉넉하지 않기 때문에

   하나의 서버에서(zookeeper) 포트만 다르게 하여 브로커의 개수를 늘리는 방식으로 분산 처리 기능을 수행하고자 한다.
2. 이미 kafka를 실행한 적이 있다면 zookeeper를 정리해줘야 한다.

   zookeeper 로그에는 snapshot이 존재한다.

   만약 이미 kafka를 실행한 적이 있다면 snapshot에는 broker가 재정의되지 않은 이전의 환경 상태가 저장되어 있을 것이다.

   따라서 zookeeper 로그를 삭제하거나 다른 공간에 로그를 path를 설정해야 한다.
   * 경로 확인

     ![image.png](/uploads/3827b7cb258f2103c8f3b6020ead0a7e/image.png)

     ![image.png](/uploads/c7a0cdba252dd29c0ce42a651e7b8c28/image.png)

     dataDir이 바로 zookeeper 로그가 저장된 경로이다. 이 위치의 내용을 모두 삭제하거나 새로운 공간에 log경로를 추가한다.
3. broker server port 추가

   broker서버는 config 폴더 아래에 server.properties를 물고 bin 폴더 아래에 kafka-server-start.sh 를 실행시켜 동작한다.

   따라서 broker server를 추가하려면 다음과 같이 server.properties를 추가하여야 한다.

   ![image.png](/uploads/7894eb166b99f3ef7b623ca5b3e61f47/image.png)

   ![image.png](/uploads/34dadce02a49e6a6398965e55045efa1/image.png)

   각각 server(2,3)properties(이하 server로 통일)는 다음 부분을 수정해줘야 한다.

   ![image.png](/uploads/dff117beebf75bcaab9cdf338a0bb702/image.png)

   우선 위와 같이 id를 수정한다. server에 따라 broker.id를 다르게 해줘야 한다. (ex-\> server2.properties는 broker.id=2로 하는 방식)

   ![image.png](/uploads/6a22511330c2c943e5a9e2a2e5ba8a3f/image.png)

   또 다음과 같이 포트를 수정한다. 포트 역시 server에 따라 다르게 해줘야한다. (ex -\> server2.properties는 :9092로 하는 방식)

   ![image.png](/uploads/ecdd51d47f0472e6479c2c8f9eae6e07/image.png)

   다음으로 log 파일의 위치를 수정한다. 로그 파일 역시 server에 따라 다르게 해줘야 한다. (ex -\> server2.properties는 /broker2로 하는 방식)

   ![image.png](/uploads/0c068faae129cd29a11e82b29f86776d/image.png)

   이 부분은 모든 server에서 통일한다.

   트랙잭션 토픽에 대한 최소 복제본의 수를 3개로 늘리고 디스크로 보내기 전 메세지의 개수를 2개로 넉넉하게 설정한다.
4. 이제 zookeeper와 각각의 서버를 실행한다.

   ![image.png](/uploads/fa292dbaa60bfdc3d009d4e6df427cf2/image.png)

   ![image.png](/uploads/6e22c64ea173387c8828006c80ff6ed1/image.png)

   ![image.png](/uploads/c07ee6a5931df8f9507f65a75ad749f7/image.png)

   ![image.png](/uploads/ff84747f7856ba05a60d5ed7f3557f35/image.png)
5. topic 을 만들고 producer - consumer 가 잘 이루어지는지 확인한다.

   ![image.png](/uploads/102e83945d1102d19125e77a690c865d/image.png)

   ./kafka-topics.sh --create --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --replication-factor 3 --partitions 10 --topic test

   ![image.png](/uploads/93454830291a3ebd2a7e4b740080856e/image.png){width="828" height="24"}./kafka-topics.sh --describe --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test

   ![image.png](/uploads/d2222ca3945bfdf925b01fc06bd89386/image.png)./kafka-console-producer.sh --broker-list 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test

   ![image.png](/uploads/ef37239f4b26d61309f327fee571fd0d/image.png)

   ./kafka-console-consumer.sh --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test --from-beginning

   ![image.png](/uploads/8cf27602e1ab6ff9d45be7d319e8ae08/image.png)
6. ui에서도 확인한다.

   ![image.png](/uploads/50aaa47285bf3872948df171933bc729/image.png)
