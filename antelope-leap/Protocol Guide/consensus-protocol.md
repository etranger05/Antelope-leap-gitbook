# 합의 프로토콜(Consensus Protocol)

## 개요

EOSIO 블록체인은 분산 방식으로 작동할 수 있는 매우 효율적이고 결정적이며 분산된 상태 머신입니다. 블록체인은 교환된 일련의 블록 내에서 트랜잭션을 추적합니다. 각 블록은 동일한 체인을 따라 이전 블록에 암호화되어 커밋됩니다. 따라서 연속 블록의 암호화 검사를 중단하지 않고 지정된 블록에 기록된 트랜잭션을 수정하는 것은 어렵습니다. 이 간단한 사실은 블록체인 트랜잭션을 변하지 않고 안전하게 만듭니다.

### 블록 생산자(Block Producers)

EOSIO 생태계에서 블록 제작 및 블록 검증은 "블록 생산자"라는 특수 노드에 의해 수행됩니다. 생산자는 EOSIO 이해관계자에 의해 선출됩니다(4. 생산자 투표/스케줄링 참조). 각 생산자는 노드 서비스를 통해 EOSIO 노드의 인스턴스를 실행합니다. 이러한 이유로, 블록을 생산하기 위해 활성 일정에 있는 생산자는 "활성(Active)" 또는 "생산(Producing)" 노드라고도 불립니다.

### 합의를 위해 필요한 사항

블록 유효성 검사는 모든 분산 노드 그룹 간에 문제를 제기합니다. 분산 시스템 내에서 이러한 블록을 무장애 방식으로 검증하려면 합의 모델이 있어야 합니다. 이러한 분산 노드와 사용자가 블록체인의 현재 상태에 합의하는 방법이 합의, 컨센서스입니다(3. EOSIO 컨센서스(DPoS + aBFT) 참조).

## 합의 모델

분산형 시스템에서 분산된 집단 그룹 간의 합의에 도달할 수 있는 다양한 방법이 있습니다. 대부분의 합의 모델은 몇 가지 증거를 통해 합의에 도달합니다. Proof of Activity(PoW와 PoS 간의 하이브리드), 화상 증명, 용량 증명, 경과 시간 증명 등과 같은 다른 유형의 증명 기반 체계가 존재하지만 가장 인기 있는 두 가지 방법은 작업 증명(PoW) 과 지분 증명(PoS)입니다. Paxos 나 Raft 같은 다른 합의 체계도 존재합니다. 이 문서는 주로 EOSIO 합의 모델에 초점을 맞추고 있습니다.

### 작업증명(Proof of Work, PoW)

블록체인에 사용되는 가장 일반적인 합의 모델 중 두 가지는 작업 증명과 지분 증명입니다. 작업 증명에서, 블록 마이닝 노드는 블록이 원하는 속성(일반적으로 블록 헤더의 암호화 해시의 가장 중요한 비트에 있는 특정 수의 0)을 갖도록 하는 블록의 헤더에 추가된 nonce 를 찾기 위해 경쟁합니다. 블록을 유효하게 만드는 그런 nonce 를 찾는 것이 계산적으로 비싸게 만들어 공격자가 네트워크의 나머지 부분에서 최고의 체인으로 받아들일 수 있는 블록체인의 대체 포크를 만드는 것이 어려워집니다. 작업 증명(Proof of Work)의 주요 단점은 네트워크의 보안이 컴퓨팅 성능에 많은 리소스를 소비하여 nonce 를 찾는 데 의존한다는 것입니다.

### 지분증명(Proof of Stake, PoS)

지분증명 알고리즘에서는 일부 자산의 가장 큰 지분이나 비율을 소유한 노드는 동등한 의사 결정 권한을 가집니다. 즉, 의결권이 보유 지분에 비례합니다. 한 가지 흥미로운 변형은 다수의 참가자 또는 이해 관계자가 더 적은 수의 대의원을 선출하고, 이에 따라 결정을 내리는 위임지분증명(Delegated Proof of Stake, DPoS)입니다.

## Antelope 의 합의 알고리즘(DPoS + aBFT)

EOSIO 기반 블록체인은 위임된 지분 증명(DPoS)을 사용하여 네트워크에서 유효한 블록에 서명할 권한이 있는 활성 생산자를 선출합니다. 그러나 이는 EOSIO 합의 프로세스의 절반에 불과합니다. 나머지 절반은 각 블록이 최종(불가역)이 될 때까지 각 블록을 확인하는 실제 프로세스에 관여하며, 이는 비동기 비잔틴 무장애(aBFT) 방식으로 수행됩니다. 따라서 EOSIO 합의 모델에는 두 가지 계층이 포함됩니다.

* 계층 1 - 기본 합의 모델(aBFT)&#x20;
* 계층 2 - 위임지분증명 (DPoS)

EOSIO에 사용되는 실제 네이티브 컨센서스 모델에는 위임/투표, 지분 또는 토큰의 개념이 없습니다. DPoS 계층은 블록 생산자의 첫 번째 일정을 생성하는 데 사용되며, 해당될 경우 각 생산자가 사이클을 완료한 후 최대 모든 일정 라운드에서 세트를 업데이트합니다. 이 두 계층은 EOSIO 소프트웨어에서 기능적으로 분리됩니다.

### 계층 1: 기본 합의 모(aBFT)

이 계층은 궁극적으로 선택된 생산자들 사이에서 수신되고 동기화되는 블록이 최종적으로 최종이 되고, 따라서 블록체인에 영구적으로 기록되는 블록을 결정합니다. 두 번째 계층에서 제안한 생산자 일정을 얻습니다(3.2 참조). 계층 2: 위임된 PoS) 및 해당 일정을 사용하여 적절한 생산자에 의해 올바르게 서명된 블록을 결정합니다. 비잔틴 내결함성의 경우 계층은 현재 예약된 세트의 생산자 중 2/3가 각 블록을 두 번 확인하는 2단계 블록 확인 프로세스를 사용합니다. 첫 번째 확인 단계는 마지막 비가역 블록(LIB)을 제안합니다. 두 번째 단계는 제안된 LIB를 최종으로 확인합니다. 이 시점에서 블록은 되돌릴 수 없게 됩니다. 이 레이어는 또한 모든 일정 라운드의 시작 시 생산자 일정 변경 신호를 보내는 데 사용됩니다.

#### Antelope 알고리즘의 최종성

EOSIO 합의 모델은 특정 블록에 서명할 권한이 있는 당사자를 결정하기 위해 일정으로 배열된 선택된 특수 참가자(활성 생산자) 집합의 서명을 통해 알고리듬 최종성을 달성합니다(작업 증명 모델에서 기껏해야 달성할 수 있는 확률론적 최종성과는 다릅니다). 시간 간격입니다. 이 일정에 대한 변경은 EOSIO 블록체인에 실행되는 특권 스마트 계약에 의해 시작될 수 있지만, 시작된 일정에 대한 변경은 두 단계의 확인에 의해 스케줄 변경을 시작한 블록이 완료되기 전까지는 적용되지 않습니다. 각 확인 단계는 현재 예정된 능동적 생산자 집합의 다수 생산자에 의해 수행됩니다.

### 계층2: 위임지분증명 (DPoS)

위임된 PoS 계층은 토큰, 지분, 투표/대리, 투표 감소, 투표 집계, 생산자 순위 및 인플레이션 보수의 개념을 소개합니다. 이 계층은 프로듀서 투표로 생성된 순위에서 새로운 프로듀서 일정을 생성하는 역할도 담당합니다. 이는 블록 생산자가 블록을 생성하고 서명하는 데 걸리는 시간인 약 2분(126초)의 일정 라운드에서 발생합니다. 타임슬롯은 제작자 1인당 총 6초 동안 진행되며, 제작자 라운드에서는 최대 12개의 블록을 제작하고 서명할 수 있습니다. DPoS 계층은 WASM 스마트 계약에 의해 활성화됩니다.

#### 스테이크 홀더와 위임

실제 활동 중인 제작자 선정(제작자 일정)은 매 일정 라운드 투표가 가능하며 참여권을 행사하는 모든 EOSIO 이해관계자가 참여합니다. 하지만 실제로, 활동적인 제작자들의 순위는 자주 바뀌지 않습니다. 이해 당사자는 EOSIO 계정 소유자로, DPoS 대표로서 자신의 블록 생산자 선호에 투표합니다. 그러나 일반 DPoS에서 크게 벗어나는 것은 일단 당선되면 얻은 표의 순위에 관계없이 모든 블록 생산자가 동등한 권한을 갖게 된다는 것입니다. 다른 DPoS 모델에서 투표권은 각 대의원이 획득한 투표 수에 비례합니다.

### 합의 프로세스

EOSIO 합의 프로세스는 두 부분으로 구성됩니다.

* 생산자 투표/스케줄링 - DPoS 계층 2에 의해 수행됩니다.&#x20;
* 블록 생산/검증 - 네이티브 컨센서스 계층 1에 의해 수행됩니다.&#x20;

이 두 프로세스는 블록체인의 첫 번째 제네시스 블록이 생성될 때 부팅 시퀀스 후 첫 번째 일정 라운드를 제외하고 독립적이며 병렬로 실행될 수 있습니다.

## 블록 생산자 투표 및 일정

다음 일정에 포함될 활성 생산자의 투표는 DPoS 계층에 의해 시행됩니다. 엄밀히 말하면, 토큰 보유자는 먼저 일부 토큰을 보유해야 이해 당사자가 되고, 따라서 주어진 이해력을 가지고 투표할 수 있습니다.

### 투표 프로세스

각 EOSIO 이해관계자는 한 번의 투표 행동으로 최대 30개의 블록 생산자에게 투표할 수 있습니다. 그런 다음 선출된 상위 21명의 생산자가 DPoS 대표로서 이해관계자를 대신하여 블록을 제작하고 서명합니다. 나머지 제작자들은 득표순으로 대기자 명단에 올려집니다. 투표 과정은 각 생산자가 얻은 투표 수를 합산하여 모든 일정 라운드를 반복합니다. 비록 투표 부패로 인해 평가절하되었지만, 생산자들은 그들의 오래된 표를 유지하는 것에 투표하지 않았습니다. 또한 투표한 제작자는 각 유권자의 마지막 투표 가중치를 새로운 투표 가중치로 대체한 것을 제외하고 이전 표를 유지할 수 있습니다.

#### 투표 가중치(Voting Weight)

각 이해관계자의 투표 가중치는 2000년 1월 1일로 정의된 EOSIO 블록 타임스탬프 에포크(epoch) 이후 경과된 시간 및 보유 토큰의 수에 대한 함수로 계산됩니다. 현재 구현에서 투표 가중치는 보유 토큰 수에 정비례하고 베이스-2는 2000년 이후 경과된 시간에 기하급수적으로 비례합니다. 실제 무게는 2^{1/52} = 1.0134192의 비율로 증가합니다. 1/52 =1.013419입니다. 이는 투표 비중이 매주 바뀌며 매년 같은 양의 토큰을 보유할 경우 두 배가 된다는 것을 의미합니다.

#### 투표 감쇠(Vote Decay)

투표 비중을 높이면 각 생산자가 보유한 현재 투표의 감가상각이 발생합니다. 이러한 투표 감쇠는 의도적이며 그 이유는 두 가지입니다.

새로운 투표가 이전 투표보다 더 많은 비중을 차지하도록 허용함으로써 참여를 장려합니다. 중요한 거버넌스 문제에 적극적으로 참여하는 사용자에게 더 많은 발언권을 부여합니다.

### 블록 생산자 일정

제작자가 투표로 결정되어 다음 일정에 선정되면, 제작자 이름별로 알파벳 순으로 간단하게 정렬됩니다. 이것은 생산 순서를 결정합니다. 각 생산자는 시작하려는 현재 일정 라운드에서 검증될 첫 번째 블록 내에서 다음 일정 라운드에 대해 제안된 생산자 세트를 받습니다. 제안된 일정을 포함하는 첫 번째 블록이 다수의 생산자와 하나의 생산자에 의해 되돌릴 수 없는 것으로 간주되면, 제안된 일정은 다음 일정 라운드에서 활성화됩니다.

#### 생산 매개변수

EOSIO 블록 제작 스케줄은 선정된 제작자들끼리 균등하게 나누어져 있습니다. 생산자는 다음 매개 변수(스케줄 라운드당)를 기준으로 각 스케줄 라운드마다 예상 블록 수를 생성할 예정입니다.

| Parameter                | Description                              | Default | Layer |
| ------------------------ | ---------------------------------------- | ------- | ----- |
| **P** (producers)        | number of active producers               | 21      | 2     |
| **Bp** (blocks/producer) | number of contiguous blocks per producer | 12      | 1     |
| **Tb** (s/block)         | Production time per block (s: seconds)   | 0.5     | 1     |

Bp(생산자당 연속 블록 수)와 Tb(블록당 생산 시간)는 계층 1 합의 상수임을 언급하는 것이 중요합니다. 대조적으로, P(활성 생산자 수)는 DPoS 계층에 의해 구성된 레이어 2 상수이며, 이는 WASM 계약에 의해 활성화됩니다.

다음 변수는 위의 매개 변수에서 정의할 수 있습니다(스케줄 라운드당).

| Variable            | Description                  | Equation                             |
| ------------------- | ---------------------------- | ------------------------------------ |
| **B** (blocks)      | Total number of blocks       | Bp (blocks/producer) x P (producers) |
| **Tp** (s/producer) | Production time per producer | Tb (s/block) x Bp (blocks/producer)  |
| **T** (s)           | Total production time        | Tp (s/producer) x P (producers)      |

따라서 계층 2에서 정의되는 P의 값은 EOSIO 블록체인에서 동적으로 변경될 수 있습니다. 그러나 실제로 N은 전략적으로 21명의 생산자로 설정되어 있으며, 이는 15명의 생산자가 2/3의 생산자와 1명의 생산자가 합의에 도달하기 위해 필요하다는 것을 의미합니다.

#### 생산 기본값

현재 기본값: P=21 선출 생산자, Bp=12 블록당 생성, T=0.5초마다 생성되는 블록의 경우, 현재 생산 시간은 다음과 같습니다(스케줄 라운드당).

| Variable                             | Value                                                           |
| ------------------------------------ | --------------------------------------------------------------- |
| **Tp**: Production time per producer | Tp = 0.5 (s/block) x 12 (blocks/producer) ⇒ Tp = 6 (s/producer) |
| **T**: Total production time         | T = 6 (s/producer) x 21 (producers) ⇒ T = 126 (s)               |

블록이 지정된 시간 슬롯 동안 지정된 생산자에 의해 생성되지 않으면, 블록 체인에 공백이 발생합니다.

## 블록 수명주기

블록은 할당된 시간 슬롯 동안 스케줄에 따라 활성 생산자에 의해 생성된 다음 동기화 및 검증을 위해 다른 생산자 노드로 릴레이됩니다. 이 과정은 생산자에서 생산자로 이어지며, 생산자의 새로운 일정이 추후 일정 라운드에서 승인될 때까지 계속됩니다. 유효한 블록이 합의 요구 사항을 충족하면(3. EOSIO 합의 참조), 블록은 최종 블록이 되며 되돌릴 수 없는 블록으로 간주됩니다. 따라서 블록은 수명 동안 프로덕션, 검증 및 최종이라는 세 가지 주요 단계를 거칩니다. 각 단계도 다양한 단계를 거칩니다.

### 블록 구조

블록의 상호 체인 시퀀스로서, 블록 체인 내의 기본 단위는 블록입니다. 블록에는 블록 확인에 필요한 해시 및 서명, 유효성 검사 중 트랜잭션 재실행, 블록체인 재생, 재생 공격 방지 등과 같은 사전 검증된 트랜잭션과 추가 암호화 오버헤드에 대한 기록이 포함됩니다(아래 블록 스키마 참조).

**블록 스키마**

| Name                 | Type                           | Description                                                                                                                       |
| -------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `timestamp`          | `block_timestamp_type`         | expected time slot this block is produced (ends in .000 or .500 seconds)                                                          |
| `producer`           | `name`                         | account name for producer of this block                                                                                           |
| `confirmed`          | `uint16_t`                     | number of prior blocks confirmed by the producer of this block in current producer schedule                                       |
| `previous`           | `block_id_type`                | block ID for previous block                                                                                                       |
| `transaction_mroot`  | `checksum256_type`             | merkle tree root hash of transaction receipts included in block                                                                   |
| `action_mroot`       | `checksum256_type`             | merkle tree root hash of action receipts included in block                                                                        |
| `schedule_version`   | `uint32_t`                     | number of times producer schedule has changed since genesis                                                                       |
| `new_producers`      | `producer_schedule_type`       | holds producer names and keys for new proposed producer schedule; null if no change                                               |
| `header_extensions`  | `extensions_type`              | extends block fields to support additional features (included in block ID calculation)                                            |
| `producer_signature` | `signature_type`               | digital signature by producer that created and signed block                                                                       |
| `transactions`       | array of `transaction_receipt` | list of valid transaction receipts included in block                                                                              |
| `block_extensions`   | `extension_type`               | extends block fields to support additional features (NOT included in block ID calculation)                                        |
| `id`                 | `block_id_type`                | UUID of this block ID (a function of block header and block number); can be used to query block for validation/retrieval purposes |
| `block_num`          | `uint32_t`                     | block number (sequential counter value since genesis block 0); can be used to query block for validation/retrieval purposes       |
| `ref_block_prefix`   | `uint32_t`                     | lower 32 bits of block ID; used to prevent replay attacks                                                                         |

블록 필드 중 일부는 블록이 생성될 때 미리 알려져 있으므로 블록 초기화 중에 추가됩니다. 트랜잭션 및 작업을 위한 머클 루트 해시, 블록 번호 및 블록 ID, 블록을 생성하고 서명한 생산자의 서명 등과 같은 다른 것들은 블록 최종화 중에 계산되고 추가됩니다(Network Peer Protocol: 3.1. 블록 ID 참조).

### **블록 생산**

블록 생산의 각 일정 라운드 동안 생산자는 가능한 한 많은 검증된 트랜잭션을 포함하는 Bp=12개의 연속 블록을 생성해야 합니다. 각 블록은 현재 Tb=500ms(0.5초)의 범위 내에서 생산됩니다. 각 블록을 생성하고 검증을 위해 다른 노드로 전송하기에 충분한 시간을 보장하기 위해 블록 생산 시간은 두 가지 구성 가능한 매개 변수로 더 나뉩니다.

* 최대 처리 간격: 트랜잭션을 블록으로 푸시하는 시간 창(현재 200ms로 설정됨).&#x20;
* 최소 전파 시간: 블록을 다른 노드에 전파하는 시간 창입니다(현재 300ms로 설정).&#x20;

아직 만료되지 않았거나 이전에 유효성 검사에 실패한 결과 삭제된 모든 느슨한 트랜잭션은 블록 포함 및 다른 노드와의 동기화를 위해 로컬 대기열에 보관됩니다. 블록 생산 중에 예약된 트랜잭션은 생산자에 의해 일정에 따라 적용되고 검증되며, 유효한 경우 처리 간격 내에 보류 중인 블록으로 푸시됩니다. 트랜잭션이 이 창을 벗어나면 적용되지 않고 다음 블록에 포함되도록 다시 예약됩니다. 현재 생산자에 사용할 수 있는 블록 슬롯이 더 이상 없으면 트랜잭션은 결국 (피어 투 피어 프로토콜을 통해) 다른 생산 노드에 의해 선택되고 다른 블록으로 푸시됩니다. 다음 생산자에게 넘겨주는 동안 네트워크 지연 시간을 보상하기 위해 마지막 블록(생산자 라운드의 Bp 블록)에 대한 최대 처리 간격이 약간 작습니다. 처리 간격이 종료되면 보류 중인 블록에서 더 이상의 트랜잭션이 허용되지 않으며, 블록은 검증을 위해 다른 블록 생산자에게 브로드캐스트되기 전에 최종화 단계를 거칩니다.

블록은 프로덕션 중에 적용, 완료, 서명 및 커밋과 같은 다양한 단계를 거칩니다.

#### 블록 적용(Apply Block)

Apply block은 기본적으로 생산 노드에서 수신하고 검증한 트랜잭션을 블록으로 푸시합니다. 내부적으로 이 단계에는 블록 헤더와 서명된 블록 인스턴스의 생성 및 초기화가 포함됩니다. 서명된 블록 인스턴스는 서명 필드를 사용하여 블록 헤더를 확장합니다. 이 필드는 결국 블록에 서명하는 생산자의 서명을 보관합니다. 또한 최근 EOSIO의 변경으로 헤더 확장 필드에 저장된 여러 서명을 포함할 수 있습니다.

#### 블록 마무리(Finalize Block)

생성된 블록이 서명, 커밋, 릴레이 및 검증되기 전에 완료되어야 합니다. 완료하는 동안 암호화 유효성 검사에 필요한 블록 헤더의 모든 필드가 계산되어 블록에 저장됩니다. 여기에는 액션 수신 목록과 블록에 푸시된 트랜잭션 수신 목록에 대한 머클 트리 루트 해시를 모두 생성하는 작업이 포함됩니다.

#### 블록 서명(Sign Block)

거래가 블록으로 밀리고 블록이 완성되면 블록은 프로듀서에 의해 서명될 준비가 됩니다. 여기에는 블록 헤더의 직렬화된 내용에서 서명 다이제스트를 계산하는 작업이 포함됩니다. 여기에는 블록에 포함된 트랜잭션 영수증이 포함됩니다. 블록이 생산자의 개인 키로 서명되면 서명된 블록 인스턴스에 서명 요약이 추가됩니다. 이렇게 하면 블록 서명이 완료됩니다.

#### 블록 커밋(Commit Block)&#x20;

블록이 서명되면 로컬 체인에 커밋됩니다. 이렇게 하면 블록이 가역 블록 데이터베이스로 푸시됩니다(Network Peer Protocol: 2.2.3 참조). 포크 데이터베이스). 이렇게 하면 블록이 검증을 위해 다른 노드와 동기화하는 데 사용할 수 있습니다(블록 동기화에 대한 자세한 내용은 네트워크 피어 프로토콜 참조).

### 블록 검증

블록 검증은 EOSIO 블록체인 내에서 합의에 도달하는 데 필요한 기본 작업입니다. 블록 유효성 검사 중에 생산자는 다른 피어에서 들어오는 블록을 수신하고 각 블록에 포함된 트랜잭션을 확인합니다. 블록 검증은 활성 생산자 간에 합의하기에 충분한 정족수에 도달하는 것입니다.

* 블록의 무결성 및 블록에 포함된 트랜잭션입니다.&#x20;
* 각 블록 내 트랜잭션의 결정론적, 시간별 순서입니다.&#x20;

블록 유효성 검사를 위한 첫 번째 단계는 노드가 블록을 수신할 때 시작됩니다. 이 시점에서 블록에 대해 몇 가지 안전 검사가 수행됩니다. 블록이 이미 알려진 블록에 링크되지 않거나 노드에서 이미 수신 및 처리된 블록의 블록 ID와 일치하면 블록이 삭제됩니다. 블록이 새 블록이면 처리를 위해 체인 컨트롤러에 푸시됩니다.

#### 블록 푸시(Push Block)

블록이 체인 컨트롤러에 의해 수신되면 소프트웨어는 로컬 체인 내에서 블록을 추가할 위치를 결정해야 합니다. 포크 데이터베이스 또는 줄여서 포크 DB가 이 용도로 사용됩니다. 포크 데이터베이스는 수신되었지만 아직 완료되지 않은 가역 블록이 있는 모든 분기를 보유합니다. 이를 위해 다음 단계를 수행합니다.

1. 포크 데이터베이스에 블록을 추가합니다.&#x20;
2. 현재 헤드 블록을 포함하는 메인 분기에 블록이 추가된 경우 블록을 적용합니다(5.2.1 참조). 블록 적용)&#x20;
3. 또는 블록을 다른 분기에 추가해야 하는 경우:
   1. 해당 분기가 현재 기본 분기와 비교하여 기본 분기가 되는 경우: 모든 블록을 가장 가까운 공통 상위 항목까지 되감고(그리고 프로세스에서 데이터베이스 상태를 롤백함), 다른 분기의 모든 블록을 다시 되감고 새 블록을 추가한 후 적용합니다. 그 분기가 이제 새로운 본점이 됩니다.&#x20;
   2. 그렇지 않은 경우: 포크 데이터베이스의 해당 분기에 새 블록을 추가하지만 다른 작업은 수행하지 않습니다.

블록이 포크 데이터베이스에 추가되려면 일부 블록 유효성 검사가 수행되어야 합니다. 블록 헤더 유효성 검사는 항상 포크 데이터베이스에 블록을 추가하기 전에 수행해야 합니다. 그리고 블록이 적용되어야 한다면 블록 내의 트랜잭션에 대한 일부 검증이 이루어져야 합니다. 트랜잭션의 유효성 검사 정도는 nodeos가 구성된 유효성 검사 모드에 따라 달라집니다. 전체 유효성 검사(기본 모드)와 광 유효성 검사라는 두 가지 블록 유효성 검사 모드가 지원됩니다.

#### 전체 유효성 검사(Full Validation)

전체 유효성 검사 모드에서는 적용된 모든 트랜잭션이 완전히 검증됩니다. 여기에는 트랜잭션의 서명 확인 및 권한 확인이 포함됩니다.

#### 약식 유효성 검사(Light Validation)

약식빛 유효성 검사 모드에서 신뢰할 수 있는 생산자가 서명한 블록(노드당 로컬로 구성할 수 있음)은 전체 유효성 검사 중에 수행된 트랜잭션 유효성 검사 중 일부를 건너뛸 수 있습니다. 예를 들어 서명 확인을 건너뛰고 액션에 대해 요청된 모든 인증이 유효한 것으로 가정합니다.

### 블록 최종 결정(Block Finality)

블록 최종성은 EOSIO 컨센서스의 최종 결과입니다. 이는 다수의 능동적 생산자가 합의 규칙에 따라 블록을 검증한 후에 달성됩니다(3.1 참조). 계층 1: 기본 합의(aBFT)입니다. 최종 단계에 도달한 블록은 블록체인에 영구적으로 기록되며 취소할 수 없습니다. 이와 관련하여, 체인의 마지막 비가역 블록(LIB)은 최종 블록이 된 가장 최근의 블록을 의미합니다. 따라서, 그 시점부터 블록체인에 기록된 트랜잭션은 뒤집거나, 변조하거나, 지울 수 없습니다.

#### 최종결정의 목표&#x20;

최종성의 요점은 LIB 블록 이전 및 이전까지 적용된 트랜잭션은 수정, 롤백 또는 삭제할 수 없다는 확신을 사용자에게 주는 것입니다. LIB 블록은 또한 가장 긴 분기에 관계없이 활성 노드가 어떤 분기에서 빌드할 것인지를 빠르고 효율적으로 결정하는 데 유용할 수 있습니다. 최신 LIB를 포함하지 않으면 지정된 분기가 더 길어질 수 있으며, 이 경우 가장 최근 LIB가 있는 더 짧은 분기를 선택해야 하기 때문입니다.