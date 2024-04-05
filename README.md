# kafka-multi-broker-clustering


##  1. 목적: 카프카의 성능 향상을 위해 분산 처리 기능을 추가하고자 한다.

```
   분산 처리를 수행하는 가장 단순한 방법은 서버를 늘리는 것이다.

   하지만 서버를 늘리게 되면 zookeeper가 1:1로 대응 되어야 하지만, 현재 제품의 용량 limit이 넉넉하지 않기 때문에

   하나의 서버에서(zookeeper) 포트만 다르게 하여 브로커의 개수를 늘리는 방식으로 분산 처리 기능을 수행하고자 한다.
```
## 2. 이미 kafka를 실행한 적이 있다면 zookeeper를 정리해줘야 한다.

```
   zookeeper 로그에는 snapshot이 존재한다.

   만약 이미 kafka를 실행한 적이 있다면 snapshot에는 broker가 재정의되지 않은 이전의 환경 상태가 저장되어 있을 것이다.

   따라서 zookeeper 로그를 삭제하거나 다른 공간에 로그를 path를 설정해야 한다.
```
   * 경로 확인

     ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/a540f2ab-5911-43ce-8345-58ee3fd15ca7)


     ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/19fe93ed-b0fa-4ee7-8e8f-0b0254bc12f5)


     dataDir이 바로 zookeeper 로그가 저장된 경로이다. 이 위치의 내용을 모두 삭제하거나 새로운 공간에 log경로를 추가한다.
## 3. broker server port 추가

```
   broker서버는 config 폴더 아래에 server.properties를 물고 bin 폴더 아래에 kafka-server-start.sh 를 실행시켜 동작한다.

   따라서 broker server를 추가하려면 다음과 같이 server.properties를 추가하여야 한다.

```
   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/bb36daa7-24a2-42e8-8af3-683ebc06d191)


   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/9d33eddd-8516-48d2-8cb9-517c828c402f)


   각각 server(2,3)properties(이하 server로 통일)는 다음 부분을 수정해줘야 한다.

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/1815c71d-2189-46d1-b293-3b7254f266ff)


   우선 위와 같이 id를 수정한다. server에 따라 broker.id를 다르게 해줘야 한다. (ex-\> server2.properties는 broker.id=2로 하는 방식)

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/730bc969-1fcb-40ea-88f0-195d4685f13d)


   또 다음과 같이 포트를 수정한다. 포트 역시 server에 따라 다르게 해줘야한다. (ex -\> server2.properties는 :9092로 하는 방식)

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/1d977633-6503-4edf-9b04-85c454f75579)


   다음으로 log 파일의 위치를 수정한다. 로그 파일 역시 server에 따라 다르게 해줘야 한다. (ex -\> server2.properties는 /broker2로 하는 방식)

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/d1dca441-e8a0-4c3d-ace9-8b8f5d6bd124)


   이 부분은 모든 server에서 통일한다.

   트랙잭션 토픽에 대한 최소 복제본의 수를 3개로 늘리고 디스크로 보내기 전 메세지의 개수를 2개로 넉넉하게 설정한다.
   
## 4. 이제 zookeeper와 각각의 서버를 실행한다.

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/9434fb0c-10ff-4164-90dd-faa0a0765adc)


   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/f2e6ba22-f6de-44a4-8880-a6cc8822a3a7)


   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/1157d762-e7c9-436b-bb5a-e162394a0f91)


   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/3e7e35ca-b882-4827-ad80-4ea54e1280e0)

## 5. topic 을 만들고 producer - consumer 가 잘 이루어지는지 확인한다.

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/60e775f4-54fb-4dfd-9ef2-5677b2d45b4d)


   ./kafka-topics.sh --create --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --replication-factor 3 --partitions 10 --topic test

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/c1f589e5-49f5-4e6b-b963-badb7289627d)
   
./kafka-topics.sh --describe --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/8a80a039-32d9-4e91-a502-7cc9d18e841c)
   
./kafka-console-producer.sh --broker-list 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/e8436172-61e6-4466-a141-26c095a57a23)


   ./kafka-console-consumer.sh --bootstrap-server 192.168.1.93:9091,192.168.1.93:9092,192.168.1.93:9093 --topic test --from-beginning

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/9fb83517-7e5c-4ea3-81f9-d19ba1078e8a)

## 6. ui에서도 확인한다.

   ![image](https://github.com/auspicious0/kafka-multi-broker-clustering/assets/108572025/f4c51033-11bf-44bf-b6b5-e96f53ef848b)

