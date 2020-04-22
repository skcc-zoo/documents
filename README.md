# 프로젝트 테스트 
## 로컬에서 테스트시

Gateway를 적용했기 때문에 모든 요청은 localhost:8080으로 보내면 된다.

아래 명령어는 모두 httpie 프로그램을 이용한 명령어임.

### 게이트 설치 (동물원 외부에서 호랑이 구경공간으로 이동하는 게이트)
게이트를 설치하면 게이트 설치 이벤트를 받아서 게이트가 연결하는 두개의 공간이 자동으로 생성된다.
```
http post localhost:8080/gate/rest/gates fromSpace="out" toSpace="tiger"
http post localhost:8080/gate/rest/gates fromSpace="tiger" toSpace="out"

```
위 명령의 결과로 "out" space, "tiger" space, "out -> tiger" gate(들어오는 문), "tiger" -> "out" gate(나가는 문)이 생성된다.

확인은 다음 명령어로 한다.

**게이트 설치된 것 확인**
```
http localhost:8080/gate/rest/gates
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

### 게이트로 사람 지나가는 이벤트 발생
이 동물원은 IoT 기술을 이용해 사람이 게이트를 통과하면 자동으로 gate 서비스의 api를 호출해주는 시스템을 가지고 있다. 호출되는 api는 다음과 같다.
```
http localhost:8080/gate/passed?gate=1
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

### 사육사 고용
우리 동물원은 특정 동물을 구경하는 사람수가 특정 값(10명)이 넘으면 동물이 스트레스를 받을 수 있어서 사육사를 파견하는 시스템이 있다.
사육사를 파견하기 전에 먼저 사육사를 고용해 보자.
```
http post localhost:8080/keeper/rest/keepers name="Brad Pitt"
http post localhost:8080/keeper/rest/keepers name="Tom Cruise"
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

### 사육사 파견
특정 동물 우리에 구경하는 사람이 10명이 되면 사육사를 파견한다.
1번게이트에 계속 사람을 통과 시켜서 tiger 우리의 인구수를 11명으로 만들어보자.
```
아래 명령어 9번 입력.
http localhost:8080/gate/passed gate=1 
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
