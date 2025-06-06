# Replication in Elasticsearch (Elasticsearch에서의 복제)

## 개요

복제(Replication)는 Elasticsearch에서 데이터를 중복하여 저장하는 과정으로, 데이터의 **중복성**, **고가용성**, **장애 허용**을 보장합니다.

## 샤드(Shards)와 복제본(Replicas)

### Primary Shard (기본 샤드)
Elasticsearch에서 각 인덱스는 기본 샤드로 나누어집니다. 기본 샤드는 원본 데이터 인덱싱과 저장을 처리합니다.

### Replica Shard (복제 샤드)
복제 샤드는 기본 샤드의 복사본입니다. 복제 샤드는 백업 역할을 하며, 쿼리 부하를 분산시킵니다.

## 복제 동작

### Primary-Replica 관계
각 기본 샤드는 하나 이상의 복제 샤드를 가질 수 있습니다.

### 쓰기 작업(Write Operations)
문서가 기본 샤드에 인덱스(저장)되면, 그 문서는 자동으로 모든 복제 샤드에 복제됩니다. 이를 통해 일관성을 보장합니다.

### 읽기 작업(Read Operations)
검색 쿼리는 기본 샤드나 복제 샤드에서 처리될 수 있습니다. 이렇게 읽기 부하가 분산되어 성능이 향상됩니다.

## 복제의 이점

### 고가용성(High Availability)
기본 샤드를 가지고 있던 노드가 다운되면, 해당 기본 샤드의 복제본이 기본 샤드로 승격되어 데이터에 접근할 수 있도록 보장합니다.

### 부하 분산(Load Balancing)
검색 요청은 기본 샤드나 복제 샤드로 라우팅될 수 있어 검색 성능이 향상됩니다.

### 데이터 중복(Data Redundancy)
데이터가 여러 노드에 복제되어 저장됨으로써, 노드 실패로 인한 데이터 손실을 방지할 수 있습니다.

## 복제를 고려할 때의 사항

### 노드 수(Number of Nodes)
복제 설정을 지원하기 위해 충분한 노드가 필요합니다. **한 노드는 동일 인덱스의 기본 샤드와 복제 샤드를 동시에 가지고 있을 수 없습니다.**

> **중요**: 한 노드(예시로 1층)안에 특정 책과 그 책의 복사본이 같이 있으면 안됩니다.
>
> **만약 같은 층에 있다면 해당 층이 불타버리면 원본이랑 복사본이 다 사라지기에 문제가 생길겁니다.**

### 네트워크 및 저장소 오버헤드(Network and Storage Overhead)
복제는 데이터 중복으로 인해 네트워크 트래픽과 저장 공간을 추가로 소모합니다.

### 트레이드오프(Trade-offs)
데이터 안전성(복제본 수 증가)과 자원 사용(디스크, 네트워크) 사이에는 균형을 맞춰야 합니다.
