# 프로젝트 테스트 방법
## 로컬에서
아래 명령어는 모두 httpie 프로그램을 이용한 명령어임.
```
# 게이트 설치 (동물원 외부에서 호랑이 구경공간으로 이동하는 게이트)
http post localhost:8080/gate/rest/gates fromSpace="out" toSpace="tiger"
```
위 명령의 결과로 gate, out space, tiger space 가 생성된다.

확인은 다음 명령어로 한다.

게이트 설치 확인
```
http localhost:8080/gate/rest/gates
---
# 결과
{
  "_embedded" : {
    "gates" : [ {
      "fromSpace" : "out",
      "toSpace" : "tiger",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8082/gate/rest/gates/1"
        },
        "gate" : {
          "href" : "http://localhost:8082/gate/rest/gates/1"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8082/gate/rest/gates"
    },
    "profile" : {
      "href" : "http://localhost:8082/gate/rest/profile/gates"
    }
  }
}

```


Space 생성확인
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
