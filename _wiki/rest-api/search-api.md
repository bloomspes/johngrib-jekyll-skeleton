---
layout  : wiki
title   : Elasticsearch 서버 구축 및 문서 생성과 조회하기
toc     : true
date    : 2023-05-07 16:09:00 +0900
updated : 2023-05-07 16:09:00 +0900
toc     : true
public  : true
parent  : rest-api
comment : true
regenerate: true
---

* TOC
{:toc}

Elasticsearch 검색 엔진을 활용해서, 통합 검색 API를 구현할 수 있는 환경을 세팅하고 실제로 문서 생성과 문서 조회를 간단하게 살펴보자.

## Elasticsearch 서버 띄우기


```
Elasticsearch with docker
```

혹은

```
dockerhub Elasticsearch
```
로 검색하면, 공식 레퍼런스와 Docker Hub 에서 친절히 세팅 방법을 알려준다.

나는 그 중에서도, `Elasticsearch` 공식 레퍼런스에 있는 가이드를 참조하겠다.

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

엘라스틱 클러스터링에는 엘라스틱서치 노드가 존재하는데, 나의 작업에서는 싱글 노드만 필요하므로, 아래처럼 elastic 네트워크를 만들고 도커 컨테이너를 설정한다.

```
docker network create elastic
```

```
docker run -it --rm --name elasticsearch \
    --net elastic \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    elasticsearch:8.7.1
```

클라이언트 포트는 9200번을 사용하고, 호스트로 포트포워딩을 할 때는 9300번을 사용하겠다는 의미이다.

주의 할 점은, 엘라스틱서치는 HTTP 통신을 하는 경우, SSL 인증을 처리하도록 되어있다.

보안을 꺼놓고 작업을 해야하는 경우라면 위에서도 서술했듯이, 엘라스틱서치 서버 실행시에 아래의 옵션을 넣어주면 된다.

```
-e "xpack.security.enabled=false"
```

엘라스틱 서버가 잘 작동했는지 확인해보자.

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html

위의 문서를 보면,

```
curl -X GET "localhost:9200/_cluster/health?pretty"
```
로 가져오거나, Httpie 이용자라면 아래처럼 요청하면 된다.

```
http localhost:9200/_cluster/health
```

아래의 데이터처럼 Http Status Code가 200번 이면서 응답이 내려오는 경우라면, 서버가 잘 작동하고 있다는 신호다.

```
{
  "cluster_name" : "testcluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 1,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50.0
}
```

응답이 잘 내려오고 있다면, 문서를 추가해보자.

아래의 링크를 참조해서, 색인 API를 구성할 수 있다.

문서를 새로 생성할 때는 POST 메소드를, 새로 생성한 문서를 수정할 경우에는 PUT 메소드를 사용하면 된다.

## 문서 추가하기

엘라스틱서치의 DB는 Document 단위를 기준으로 데이터를 관리하고 있다. 문서를 추가한다는 것은, DB에 데이터를 등록하는 것과 의미가 동일하다.

아래의 링크를 참조해서, 엘라스턱서치의 문서를 생성할 수 있다.

https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html


## Httpie

나는 이 작업을 할 때 주로 Httpie를 사용하는 편이다.

이유는, 터미널에서 데이터를 참 예쁘게 보여준다. 또, 실행이 매우 간편하다.

https://httpie.io/docs/cli/installation

SSL Verification 단계를 스킵하고 싶을 때는, 아래처럼 옵션을 찾아서 적용하면 된다.

https://httpie.io/docs/cli/server-ssl-certificate-verification

## 문서 검색하기

전체 문서 조회

```
http localhost:9200/<target>/_search/
```

query로 검색해서 문서 조회

```
http localhost:9200/test/_search/ <<< '
{
  "query": {
    "query_string": {
      "fields": [
        "body"
      ],
      "query": "슭"
    }
  }
}
'
```
위의 요청을 실행하면 body에 "슭"이 포함된 문서를 찾는다.











