# Fluetned & Fluentbit

## 목적
- 단일 통합 로깅 레이어
- 로그 표준화 및 json 전환
- 로그 캐싱(파일, 메모리) 및 라우팅
- 플러그인 (input / filter / output)
- Fluentbit 은 fluentd 경량 버전

## 개념
- Input
- Parser
- Filter
- Buffer
- Router
- Output

## 시험
- [docker & docker-compose](docker-compose.md) 준비
- fluentbit input cpu with stdout
```
$ sudo docker run -it fluent/fluent-bit /fluent-bit/bin/fluent-bit -i cpu -o stdout -f 1

Fluent Bit v1.6.5
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2020/11/21 07:21:12] [ info] [engine] started (pid=1)
[2020/11/21 07:21:12] [ info] [storage] version=1.0.6, initializing...
[2020/11/21 07:21:12] [ info] [storage] in-memory
[2020/11/21 07:21:12] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2020/11/21 07:21:12] [ info] [sp] stream processor started
[0] cpu.0: [1605943273.156827976, {"cpu_p"=>0.250000, "user_p"=>0.000000, "system_p"=>0.250000, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000}]
```
- fluentbit [dummy input](https://docs.fluentbit.io/manual/pipeline/inputs/dummy) with [elasticsearch output](https://fluentbit.io/documentation/0.14/output/elasticsearch.html)
```
$ cat fluent-bit.conf 
[INPUT]
  Name dummy
  Dummy {"top": {".dotted": "value"}}

[OUTPUT]
  Name es
  Host elasticsearch
  Replace_Dots On

$ cat docker-compose.yaml 
version: "3.7"

services:
  fluent-bit:
    image: fluent/fluent-bit
    volumes:
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
    depends_on:
      - elasticsearch
  elasticsearch:
    image: elasticsearch:7.6.2
    ports:
      - "9200:9200"
    environment:
      - discovery.type=single-node

$ curl "localhost:9200/_search?pretty" \
  -H 'Content-Type: application/json' \
  -d'{ "query": { "match_all": {} }}'
  
$ curl -X DELETE "localhost:9200/fluent-bit?pretty"

$ docker-compose up
$ docker-compose ps
$ docker-compose stop
$ docker-compose down
```

## fluentbit 설정 사례
- cpu & elastic search
```
$ cat fluent-bit.conf   # fluent-bit -i cpu -t cpu -o es://192.168.2.3:9200/my_index/my_type -o stdout -m '*'
[INPUT]
    Name  cpu
    Tag   cpu

[OUTPUT]
    Name  es
    Match *
    Host  192.168.2.3
    Port  9200
    Index my_index
    Type  my_type
```
- dummy & stdout
```
$ cat fluent-bit.conf     # fluent-bit -i dummy -o stdout
[INPUT]
    Name   dummy
    Tag    dummy.log

[OUTPUT]
    Name   stdout
    Match  *    
```
- monitoring
```
$ cat fluent-bit.conf 
[SERVICE]
    HTTP_Server  On
    HTTP_Listen  0.0.0.0
    HTTP_PORT    2020

[INPUT]
    Name cpu

[OUTPUT]
    Name  stdout
    Match *
    
$ curl -s http://127.0.0.1:2020/api/v1/metrics/prometheus    
```

## 참고
- https://www.fluentd.org/blog/unified-logging-layer
- https://github.com/fluent/fluent-bit
- https://docs.fluentbit.io/manual/concepts/key-concepts
- https://docs.fluentbit.io/manual/v/master/local-testing/logging-pipeline
- https://docs.fluentbit.io/manual/installation/docker
