
MS1 UNIT

10098 

원찬희 

Azure ID: admin10

# 서비스 시나리오
## 티켓 구매 
1. 매표소 직원은(pay 서비스) 티켓 구매 요청이 들어오면 space 서비스에 티켓 발행이 가능한지 문의한다. 이때 Sync방식으로 Request를 보낸다.
1. space 서비스는 현재 동물원 총 입장 인원을 확인하여 한계치(기본값 50명)를 넘지 않았으면 티켓 발행 가능 사인을, 넘었으면 티켓 발행 불가능 사인을 반환한다.
1. space 서비스의 반환 결과에 따라 티켓을 발행하거나 발행하지 않는다.

## 게이트 설치
1. 게이트는 공간과 공간이 나뉘는 경계에 설치된다고 가정한다.
1. 게이트를 설치하면 gate 서비스에 게이트 설치됨 이벤트가 발생한다.
1. 게이트 설치됨 이벤트를 space 서비스에서 수신하여, 만일 게이트가 나누는 두개의 공간 중, DB에 등록되지 않은 공간이 있다면 DB에 등록한다.
공간이 새로 등록된 경우 공간 등록됨 이벤트가 발생한다.
1. 공간 등록됨 이벤트를 state 서비스(CQRS)가 수신하여 자신의 DB에도 공간에 대한 정보를 업데이트 한다.

## 사육사 고용
1. 사육사가 고용된 경우 keeper 서비스의 API를 직접 호출하여 사육사를 등록한다. 

## 관람객 이동 관리
1. 관람객이 물리적 게이트를 통과하면 물리적 게이트에서는 gate 서비스의 API를 호출하여 관람객이 지나갔음을 알린다.
1. 호출된 gate 서비스는 관람객 지나감 이벤트를 발행한다.
1. 관람객 지나감 이벤트를 받은 space 서비스는 각 공간의 현재 인원수를 변경한다. 그리고 인원수 변경됨 이벤트를 발행한다.
1. 인원수 변경됨 이벤트를 받은 keeper 서비스는 해당 공간의 관람객 수가 정해진 기준(기본값 10명)을 넘어서는 경우 사육사를 파견한다. 그리고 사육사 파견됨 이벤트를 발생한다.
1. 사육사 파견됨 이벤트를 받은 state 서비스(CQRS)는 자신의 DB를 업데이트 한다.

## 현황 관리
1. 전체 시스템에서 발행된 이벤트를 수신하여 state 서비스에서는 각 공간에 현재 관람객이 몇명이고 어느 사육사가 파견되어 있는지를 보여준다.

# 프로젝트 테스트 방법 

Gateway를 적용했기 때문에 모든 요청은 localhost:8080으로 보내면 된다.

Cloud에 배포된 버전을 테스트 하기 위해서는 52.231.107.109:8080 으로 요청을 보내면 된다.

아래 명령어는 모두 httpie 프로그램을 이용한 명령어임.

## 게이트 설치 (동물원 외부에서 호랑이 구경공간으로 이동하는 게이트)
게이트를 설치하면 게이트 설치 이벤트를 받아서 게이트가 연결하는 두개의 공간이 자동으로 생성된다.
```
http post localhost:8080/gate/rest/gates fromSpace="out" toSpace="tiger"
http post localhost:8080/gate/rest/gates fromSpace="tiger" toSpace="out"

# cloud
http post 52.231.107.109:8080/gate/rest/gates fromSpace="out" toSpace="tiger"
http post 52.231.107.109:8080/gate/rest/gates fromSpace="tiger" toSpace="out"

```
위 명령의 결과로 "out" space, "tiger" space, "out -> tiger" gate(들어오는 문), "tiger" -> "out" gate(나가는 문)이 생성된다.

확인은 다음 명령어로 한다.

**게이트 설치된 것 확인**
```
http localhost:8080/gate/rest/gates

# cloud
http 52.231.107.109:8080/gate/rest/gates
---
# 결과
            {
                "_links": {
                    "gate": {
                        "href": "http://localhost:8082/gate/rest/gates/1"
                    },
                    "self": {
                        "href": "http://localhost:8082/gate/rest/gates/1"
                    }
                },
                "fromSpace": "out",
                "toSpace": "tiger"
            },
            {
                "_links": {
                    "gate": {
                        "href": "http://localhost:8082/gate/rest/gates/2"
                    },
                    "self": {
                        "href": "http://localhost:8082/gate/rest/gates/2"
                    }
                },
                "fromSpace": "tiger",
                "toSpace": "out"
            }

```


**Space 생성된 것 확인**
```
http localhost:8080/space/rest/spaces

#cloud
http 52.231.107.109:8080/space/rest/spaces
---
# 결과
{
  "_embedded" : {
    "spaces" : [ {
      "population" : 0,
      "_links" : {
        "self" : {
          "href" : "http://localhost:8083/space/rest/spaces/out"
        },
        "space" : {
          "href" : "http://localhost:8083/space/rest/spaces/out"
        }
      }
    }, {
      "population" : 0,
      "_links" : {
        "self" : {
          "href" : "http://localhost:8083/space/rest/spaces/tiger"
        },
        "space" : {
          "href" : "http://localhost:8083/space/rest/spaces/tiger"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8083/space/rest/spaces"
    },
    "profile" : {
      "href" : "http://localhost:8083/space/rest/profile/spaces"
    }
  }
}
```

## 게이트로 사람 지나가는 이벤트 발생
이 동물원은 IoT 기술을 이용해 사람이 게이트를 통과하면 자동으로 gate 서비스의 api를 호출해주는 시스템을 가지고 있다. 호출되는 api는 다음과 같다.
```
http localhost:8080/gate/passed?gate=1

#cloud
http 52.231.107.109:8080/gate/passed?gate=1
---
# 결과
success
```
아까 만든 1번 게이트는 out(밖)과 호랑이 우리를 잇는 게이트 이다. 이 결과 out space의 population이 -1이 되고 tiger space의 인구가 +1이 된다.
아래 명령어로 확인할 수 있다.
```
http localhost:8080/space/rest/spaces
---
# 결과
"spaces": [
            {
                "_links": {
                    "self": {
                        "href": "http://localhost:8083/space/rest/spaces/out"
                    },
                    "space": {
                        "href": "http://localhost:8083/space/rest/spaces/out"
                    }
                },
                "population": 0
            },
            {
                "_links": {
                    "self": {
                        "href": "http://localhost:8083/space/rest/spaces/tiger"
                    },
                    "space": {
                        "href": "http://localhost:8083/space/rest/spaces/tiger"
                    }
                },
                "population": 1
            }
        ]
```
위에서 out의 population은 0인데 이는 밖의 인구는 무한하다는 가정하에 항상 0 이상의 값을 갖도록 설정했기 때문이다.이 시스템에서 외부에 사람이 얼마나 있는지는 중요하지 않다.

## 사육사 고용
우리 동물원은 특정 동물을 구경하는 사람수가 특정 값(5명)이 넘으면 동물이 스트레스를 받을 수 있어서 사육사를 파견하는 시스템이 있다.
사육사를 파견하기 전에 먼저 사육사를 고용해 보자.
```
http post localhost:8080/keeper/rest/keepers name="Brad Pitt"
http post localhost:8080/keeper/rest/keepers name="Tom Cruise"

#cloud
http post 52.231.107.109:8080/keeper/rest/keepers name="Brad Pitt"
http post 52.231.107.109:8080/keeper/rest/keepers name="Tom Cruise"
---
# 결과
{
    "_links": {
        "keeper": {
            "href": "http://localhost:8084/keeper/rest/keepers/1"
        },
        "self": {
            "href": "http://localhost:8084/keeper/rest/keepers/1"
        }
    },
    "name": "Brad Pitt",
    "space": null
}
```
브래드 피트와 톰 를 고용했다. 고용 당시 space는 null인데 이는 어느 동물 우리에도 가 있지 않은 상태를 의미한다.

## 사육사 파견
특정 동물 우리에 구경하는 사람이 10명이 되면 사육사를 파견한다.
1번게이트에 계속 사람을 통과 시켜서 tiger 우리의 인구수를 11명으로 만들어보자.
```
아래 명령어 9번 입력.
http localhost:8080/gate/passed?gate=1 

# cloud
http 52.231.107.109:8080/gate/passed?gate=1
```
그리고 사육사 상태를 살펴보면
```
http localhost:8080/keeper/rest/keepers
---
# 결과
"keepers" : [ {
      "name" : "Tom Cruise",
      "space" : "tiger",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8084/keeper/rest/keepers/1"
        },
        "keeper" : {
          "href" : "http://localhost:8084/keeper/rest/keepers/1"
        }
      }
    }, {
      "name" : "Brad Pitt",
      "space" : null,
      "_links" : {
        "self" : {
          "href" : "http://localhost:8084/keeper/rest/keepers/2"
        },
        "keeper" : {
          "href" : "http://localhost:8084/keeper/rest/keepers/2"
        }
      }
    } ]
```
Tom Cruise가 호랑이 우리로 파견되었음을 알 수 있다. 2번 게이트로 사람을 지나가게 해서 호랑이 우리의 사람수가 10명 아래로 내려가면 파견된 Tom Cruise는 다시 돌아온다.

## 표 구입
우리 동물원은 수익보다도 동물의 복지를 최우선으로 하기 때문에 일정 방문객 이상은 받지 않는다. 따라서 표 구입시 결제 시스템은 공간 관리 시스템에 현재 입장권을 판매해도 되는지 물어보는 Request를 직접 날린다. 공간 관리 시스템에서 방문객이 수용가능 인원을 넘은 경우에는 입장권 판매를 거부하게 된다.
```
# 입장권 구매
http localhost:8080/pay/buy_ticket

#cloud
http 52.231.107.109:8080/pay/buy_ticket
---
#결과
동물원 관람객이 50명 미만일 때
{
    "message": "Welcome to my zoo!!",
    "price": 100000
}

동물원 관람객이 50 넘을 때
{
    "message": "Not available, now. Sorry.",
    "price": 0
}
```

## 지금까지의 현황 확인
어느 동물 우리에 몇명이 있고 어느 사육사가 파견되어있는지를 확인해보자.
```
http localhost:8080/state/

# cloud
http 52.231.107.109:8080/state/
---
# 결과
[
    {
        "keeper": null,
        "populate": -10,
        "space": "out"
    },
    {
        "keeper": "Brad Pitt",
        "populate": 10,
        "space": "tiger"
    }
```
# 분석
## 이벤트 스토밍 결과
![event-storming](https://github.com/skcc-zoo/documents/blob/master/event-storming.png)

## 도매인 서열
본 동물원은 매출보다 동물의 복지를 우선시 한다. 그래서 게이트 시스템과 공간 관리 시스템이 코어 시스템이다. 동물원 전체에 관람객이
일정수를 초과하면 사람들이 입장하지 못하도록 하는 것이 가장 중요한 기능이다.

코어 도매인: 게이트 시스템 / 공간 관리 시스템
서포팅 도매인: 사육사 관리 시스템 / 현황 관리 시스템(CQRS)
제너럴 도매인: 결제 시스템

## 헥사고날 아키텍쳐
![hexagonal-architecture](https://github.com/skcc-zoo/documents/blob/master/hexagonal-arch.png)

# 구현
- 모든 MSA는 Spring Boot로 구현하였다.
- Gateway를 이용하여 진입점을 통일하였다.
- Oauth를 적용하였으나, 수업시간에 사용한 예제를 그대로 사용한 수준이다.
- Netflix Hystrix를 사용하여 pay 서비스에 Circuit Breaker를 적용하였다.
- Pay 서비스와 Space 서비스 간에 동기식 호출을 적용하였다. 코어 시스템인 Space 서비스가 작동하지 않으면 표를 팔지 않는다.
- 나머지 다른 모든 시스템들은 이벤트 스트림을 이용하여 비동기식 호출 / 장애 격리 / 최종 일관성 테스트를 구현하였다.

# 운영
- master 브랜치 커밋시 자동으로 배포되도록 pipeline 구성.
- 서킷 브레이커 적용함.
- 모든 서비스에 Readiness Probe를 적용하여 무정지 배포 적용함.
- 코어 서비스인 Space 서비스에 오토 스케일 적용함.
- Space 서비스에 configMap 적용함. 동물원 최대 관람 인원을 configMap을 통해서 환경변수로 주입.
``` yaml
 env:
 - name: MAX_POPULATION
   valueFrom:
     configMapKeyRef:
       name: max-population-configmap
       key: MAX_POPULATION

```
- Keeper 서비스에 secret 적용함. 공간 하나당 관람 인원을 secret을 통해서 환경변수로 주입
``` yaml
env:
- name: MAX_ANIMAL_POPULATION
  valueFrom:
    secretKeyRef:
      name: max-animal-population-configmap
      key: MAX_ANIMAL_POPULATION

```
