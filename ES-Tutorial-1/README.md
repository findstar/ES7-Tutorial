# ES-Tutorial-1

ElasticSearch 첫 번째 튜토리얼을 기술합니다.

본 스크립트는 외부 공인망을 기준으로 작성되었습니다.

## ElasticSearch Product 설치

이 튜토리얼에서는 rpm 파일을 이용하여 실습합니다.

Master Node 1번 장비에서 실습합니다.

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo yum -y install git

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ git clone https://github.com/benjamin-btn/ES7-Tutorial.git

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ cd ES7-Tutorial/ES-Tutorial-1

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1

##################### Menu ##############
 $ ./tuto1 [Command]
#####################%%%%%%##############
         1 : elasticsearch packages
         2 : configure elasticsearch.yml & jvm.options
         3 : start elasticsearch process
         4 : install kibana packages
         5 : configure kibana.yml
         6 : start kibana process
         init : ec2 instance initializing
#########################################

```

## ELK Tutorial 1 - Elasticsearch, Kibana 세팅

### Elasticsearch
##### /etc/elasticsearch/elasticsearch.yml

1) cluster.name, node.name, network.host, http.cors.enabled, http.cors.allow-origin 추가설정
2) **./tuto1 1 ./tuto1 2 실행 후 cluster.name 은 unique name 으로 별도 설정 필요**
3) 7.x 부터 변경된 discovery 설정 추가

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 1

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 2

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ sudo vi /etc/elasticsearch/elasticsearch.yml


### For ClusterName & Node Name
cluster.name: mytuto-es # Your Unique Cluster Name
node.name: master-ip-172-31-14-110 # Your Unique Node Name

### For Head
http.cors.enabled: true
http.cors.allow-origin: "*"

### For Response by External Request
network.host: 0.0.0.0

### Discovery Settings
discovery.seed_hosts: [ "{IP1}:9300",  ]
cluster.initial_master_nodes: [ "{IP1}:9300",  ]

```

##### /etc/elasticsearch/jvm.options
3) Xms1g, Xmx1g 를 물리 메모리의 절반으로 수정

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ sudo vi /etc/elasticsearch/jvm.options

-Xms2g
-Xmx2g

```

4) 두 파일 모두 수정이 완료되었으면 ./tuto1 3 을 실행하여 ES 프로세스 시작, 클러스터가 잘 구성되었는지 확인

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 3

```


### Kibana
/etc/kibana/kibana.yml
1) server.host 를 외부에서도 접근 가능하도록 0.0.0.0 으로 설정
2) elasticsearch.url 은 localhost 에 ES 도 함께 설치했기 때문에 http://localhost:9200 으로 설정
3) kibana.index 는 기본이름인 ".kibana" 로 설정

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1

##################### Menu ##############
 $ ./tuto1 [Command]
#####################%%%%%%##############
         1 : install java & elasticsearch packages
         2 : configure elasticsearch.yml & jvm.options
         3 : start elasticsearch process
         4 : install kibana packages
         5 : configure kibana.yml
         6 : start kibana process
         init : ec2 instance initializing
#########################################

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 4

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 5

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 6

```

## Smoke Test

### Elasticsearch

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ curl localhost:9200
{
  "name" : "master-ip-172-31-13-126",
  "cluster_name" : "mytuto-es",
  "cluster_uuid" : "LTfRfk3KRLS31kQDROVu9A",
  "version" : {
    "number" : "7.3.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "a9861f4",
    "build_date" : "2019-01-24T11:27:09.439740Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ curl -H 'Content-Type: application/json' -XPOST localhost:9200/firstindex/_doc -d '{ "mykey": "myvalue" }'
```

* Web Browser 에 [http://ec2-52-221-155-168.ap-southeast-1.compute.amazonaws.com:9100/index.html?base_uri=http://{FQDN}:9200](http://ec2-52-221-155-168.ap-southeast-1.compute.amazonaws.com:9100/index.html?base_uri=http://FQDN:9200) 실행

![Optional Text](image/es-head.png)

### Kibana
* Web Browser 에 [http://{FQDN}:5601](http://{FQDN}:5601) 실행

![Optional Text](image/kibana.png)

## Trouble Shooting

### Elasticsearch
Smoke Test 가 진행되지 않을 때에는 elasticsearch.yml 파일에 기본으로 설정되어있는 로그 디렉토리의 로그를 살펴봅니다.

path.logs: /var/log/elasticsearch 로 설정되어 cluster.name 이 적용된 파일로 만들어 로깅됩니다.

위의 경우에는 /var/log/elasticsearch/mytuto-es.log 에서 확인할 수 있습니다.

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ sudo vi /var/log/elasticsearch/mytuto-es.log
```
  
로그도 남지 않았으면 elasticsearch user 로 elasticsearch binary 파일을 직접 실행해 로그를 살펴봅시다

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ sudo -u elasticsearch /usr/share/elasticsearch/bin/elasticsearch
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
[2020-01-21T16:22:17,706][INFO ][o.e.e.NodeEnvironment    ] [master-ip-172-31-5-69] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [6gb], net total_space [9.9gb], types [rootfs]
[2020-01-21T16:22:17,708][INFO ][o.e.e.NodeEnvironment    ] [master-ip-172-31-5-69] heap size [1.9gb], compressed ordinary object pointers [true]
[2020-01-21T16:22:17,740][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [master-ip-172-31-5-69] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.IllegalStateException: Node is started with node.data=false, but has shard data: [/var/lib/elasticsearch/nodes/0/indices/se6EbqfOQieAKo_CoASbOg/0, /var/lib/elasticsearch/nodes/0/indices/MFk046FfS0CWeXWJscO-MQ/0]. Use 'elasticsearch-node repurpose' tool to clean up
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:125) ~[elasticsearch-cli-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-7.5.1.jar:7.5.1]
Caused by: java.lang.IllegalStateException: Node is started with node.data=false, but has shard data: [/var/lib/elasticsearch/nodes/0/indices/se6EbqfOQieAKo_CoASbOg/0, /var/lib/elasticsearch/nodes/0/indices/MFk046FfS0CWeXWJscO-MQ/0]. Use 'elasticsearch-node repurpose' tool to clean up
	at org.elasticsearch.env.NodeEnvironment.ensureNoShardData(NodeEnvironment.java:1081) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.env.NodeEnvironment.<init>(NodeEnvironment.java:325) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.node.Node.<init>(Node.java:273) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.node.Node.<init>(Node.java:253) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:221) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:221) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:349) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-7.5.1.jar:7.5.1]
	... 6 more
```
