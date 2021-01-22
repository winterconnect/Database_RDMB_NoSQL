# NoSQL

### 

- 관계형 데이터베이스 기술의 문제점: 클라우드 컴퓨팅 환경에서 발생하는 빅데이터를 효과적으로 저장, 관리하기 어려움
- 이 문제점을 개선, 보완하기 위해 NoSQL 등장
- 분산 시스템, 병렬 시스템 지원
- 명령어들이 RDBMS와 조금씩 다름
- 안정성은 떨어질 수 있음(오픈소스이므로)



### NoSQL 의 장점

1. __클라우드 컴퓨팅 환경에 적합__

   1) 오픈소스

   2) 하드웨어 확장에 유연한 대처가 가능

   3) RDBMS에 비해 저렴한 비용으로 분산처리와 병렬처리 가능

2. __유연한 데이터 모델__

   1) 비정형 데이터 구조 설계로 설계 비용 감소

   2) linking과 embedded로 RDBMS의 relation,join 구조 구현

3. __빅데이터 처리에 효과적__

   1) mermory mapping기능을 통해 read/write가 빠름

   2) 전형적인 os와 하드웨어에 구축 가능

   3) 기존 RDB와 동일하게 데이터 처리가 가능



참고) map reduce algorithm

구글의 map reduce algorithm을 보고 아파치에서 하둡을 만들고 경쟁구도 만들어짐

(사실상 구글이 우세)



### NoSQL의 제품군

1. Key-Value Database: collection of K-V pairs

2. Bigtable Database: NETFLIX 

3. document database

4. graph database



### CAP이론

__Consistency(일관성)__: 데이터가 변하지 않고 일관적이어야 함

__Availability(가용성)__: 쓰고싶을 때 쓸 수 있어야 함

__Partition(지속성)__





## Hadoop

### Hadoop Ecosystem

__1. HDFS(Hadoop Distributed File System)__

- File System
  - 하드디스크에 파일을 저장하는 방식(윈도우냐, 리눅스냐 등으로 나뉨)
  - os형태에 따라서 다른 파일시스템이 들어온다
  - 윈도우는 NTFS, 리눅스는 ext4(node 형식)

- 구글은 GFS(google file system)로 파일을 만들었음(NTFS 가 속도가 느리고 최적화되어있지 않음)

- 아파치 진영의 하둡은 리눅스 os를 기반으로 HDFS(하둡 분산 파일 시스템)을 만듦



__2. Map Reduce__: 구글에서 발표한 알고리즘

- Map에서 슬라이싱
  - input 데이터(K-V)를 여러개의 split-piece로 분산입력
  - ex) 1~100 까지 그룹핑, 100~1000까지 그룹핑, 1000~10000까지 그룹핑
  - 그룹마다 부분합이 나오게 됨

- Reduce에서 부분합을 aggregate하는 알고리즘
  - 중복데이터를 제거한 후 사용자가 원하는 형태로 데이터 집계
- 구글 MapReduce를 기반으로 Hadoop MapReduce가 설계되었으며 Hadoop HDFS를 이용한 분산파일 시스템
- 실제 속도가 매우 빠름





### Hadoop Architecture

__1. 데이터 저장 방식: 클라우드 서버, Node 방식__

- Master Node 서버 (+ master node backup 서버)
- Data Node 서버(여러개 서버로 분산)



__Node 저장순서__

- Client가 데이터를 저장하고 싶으면, master node에 물어봄.
- master node가 data node에 물어봄: 공간있어?
- data node가 대답함: 어디(주소)에 저장할 수 있어!
- master node가 client에게 주소 전달 (data node중 하나)
- data node에서 데이터를 다른 data node들에 복사하여 전달
  - 하나의 node에서 데이터가 손실되더라도 다른 node를 통해 복구됨

- __단점__: 네트워크가 안되면 사용 불가(그러나 모든 네트워크가 끊기는 경우는 거의 없다)





## MongoDB

- JSON 타입의 데이터 저장 구조 제공
- Sharding(분산), Replica(복제) 기능 제공
- CRUD(Create, Read, Update, Delete) 위주의 다중 트랜잭션 처리 가능
- Memory Mapping 기술 기반으로 빅데이터 처리에 탁월한 성능 제공



### 설치

```
brew install node
brew install mongodb-community
mongod --version

mongo localhost:27017
```





db.help()

db.version()

db.stats()

```
>db.stats()
 {
        "db" : "show",          <-DB명
        "collections" : 0,      <-컬렉션 수
        "views" : 0,            <-뷰어의 수
        "objects" : 0,          <-오브젝트 수
        "avgObjSize" : 0,       <-오브젝트의 평균 크기
        "dataSize" : 0,         <-데이터 크기
        "storageSize" : 0,      <-저장공간의 크기
        "numExtents" : 0,       <- 총 익스턴트 수
        "indexes" : 0,          <- 인덱스 수
        "indexSize" : 0,        <- 인덱스 크기
        "fsUsedSize" : 0,       <- 사용된 파일 크기
        "fsTotalSize" : 0,      <- 총 파일 크기
        "ok" : 1                <- 문장의 실행 상태(정상:1,  실패:0) 
}
```



db.hostInfo()

```
> db.hostInfo()
{
	"system" : {
		"currentTime" : ISODate("2021-01-22T04:21:16.895Z"),
		"hostname" : "Winterui-MacBookPro.local",   <- 호스트명
		"cpuAddrSize" : 64,
		"memSizeMB" : NumberLong(8192),             <- 메모리 크기
		"memLimitMB" : NumberLong(8192),
		"numCores" : 4,                              <- 코어수
		"cpuArch" : "x86_64",
		"numaEnabled" : false
	},
	"os" : {
		"type" : "Darwin",
		"name" : "Mac OS X",
		"version" : "19.6.0"
	},
	"extra" : {
		"versionString" : "Darwin Kernel Version 19.6.0: Thu Oct 29 22:56:45 PDT 2020; root:xnu-6153.141.2.2~1/RELEASE_X86_64",
		"alwaysFullSync" : 0,
		"nfsAsync" : 0,
		"model" : "MacBookPro12,1",
		"physicalCores" : 2,
		"cpuFrequencyMHz" : 2700,
		"cpuString" : "Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz",
		"cpuFeatures" : "FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE SSE3 PCLMULQDQ DTES64 MON DSCPL VMX EST TM2 SSSE3 FMA CX16 TPR PDCM SSE4.1 SSE4.2 x2APIC MOVBE POPCNT AES PCID XSAVE OSXSAVE SEGLIM64 TSCTMR AVX1.0 RDRAND F16C SYSCALL XD 1GBPAGE EM64T LAHF LZCNT PREFETCHW RDTSCP TSCI",
		"pageSize" : 4096,
		"scheduler" : "dualq"
	},
	"ok" : 1
}
```



### __db.ShutdownServer()와 db.logout()의 차이점__

> mongod --dbpath (인스턴스 활성화)
>
> mongod (클라이언트 접속)



인스턴스를 활성화한 후 여러개 클라이언트 실행(mongod 여러 터미널 창)했다고 가정하면, 

__db.shutdownServer()__: 서버 자체가 끊어짐, 인스턴스가 비활성화됨 (DB종료)

__db.logout()__ : 인스턴스 활성화는 유지하지만 클라이언트만 로그아웃됨







## MongoDB 데이터 처리

### Terminology

- RDBMS와 비교

| RDBMS                            | NoSQL                                                 |
| -------------------------------- | ----------------------------------------------------- |
| Primary Key                      | _ID Field                                             |
| Table<br />CREATE TABLE 테이블명 | Collection<br />db.createCollection(컬렉션명, 설정값) |
| Row                              | BSON Document                                         |
| Column                           | BSON Field                                            |
| Relationship                     | Embedded & Linking                                    |





### Collection의 생성/수정/삭제

- Capped Collection 과 Non Capped Collection 두가지 종류



__use SALES__

```
> use SALES
switched to db SALES
```

- RDBMS에서는 DB가 없으면 에러가 나지만, SALES 데이터베이스가 없어도 만들어짐



__db.createCollection()__

```sql
> db.createCollection ("employees" , {capped: true, size: 100000 })
{ "ok" : 1 }
-- capped: 저장공간의 재생이 가능한 타입
-- size: collection의 최초 익스텐트 크기
```



__db.employees.stats()__



__db.employees.insert({key:value})__



__db.employees.find()__



__db.employees.renameCollection("emp")__: Collection 이름 변경

```sql
-- emp로 이름 변경
> db.employees.renameCollection("emp")
{ "ok" : 1 }
```



__db.emp.drop()__: Collection  삭제

```
> db.emp.drop()
true
```





### 데이터 입력/수정/삭제

- k-v 쌍으로 저장되기 때문에 길이가 달라도 저장되며, 비정형데이터도 무리없이 가능!

```sql
> m={ename : "smith"} <- MongoDB에서는 JSON 타입으로 데이터를 표현
> n={ empno : 1101 }
```



```sql
> db.things.save(m)  <- 데이터를 저장할때 SAVE 함수를 사용
> db.things.save(n)
> show collections
things
-- collection을 만들지 않아도 자동으로 collection이 생성됨
```



```sql
> db.things.find()    <- Collection에 저장된 데이터를 검색할 때 FIND 메소드 실행
{ "_id" : ObjectId("600a6d5a525efb0c370f02bd"), "ename" : "smith" }
{ "_id" : ObjectId("600a6d61525efb0c370f02be"), "empno" : 1101 }
```



__db.things.insert({k:v});__

```
db.things.insert({ empno : 1102, ename : "king"})
```



__실험__: 1101을 하나 더 insert 한다면?

```sql
> db.things.insert({empno : 1101, job : "student"})

> db.things.find()
{ "_id" : ObjectId("600a6d5a525efb0c370f02bd"), "ename" : "smith" }
{ "_id" : ObjectId("600a6d61525efb0c370f02be"), "empno" : 1101 }
{ "_id" : ObjectId("600a6ff9525efb0c370f02bf"), "empno" : 1102, "ename" : "king" }
{ "_id" : ObjectId("600a70c9525efb0c370f02c0"), "empno" : 1101, "job" : "student" }

-- insert 로 넣으면 empno가 같아도 전혀 다른 값으로 인식함
```



__db.things.save()__

```sql
> for (var n = 1103; n <= 1120; n++) db.things.save({n : n , m : "test"})
{ "_id" : ObjectId("600a6d5a525efb0c370f02bd"), "ename" : "smith" }
{ "_id" : ObjectId("600a6d61525efb0c370f02be"), "empno" : 1101 }
{ "_id" : ObjectId("600a6ff9525efb0c370f02bf"), "empno" : 1102, "ename" : "king" }
{ "_id" : ObjectId("600a70c9525efb0c370f02c0"), "empno" : 1101, "job" : "student" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c1"), "n" : 1103, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c2"), "n" : 1104, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c3"), "n" : 1105, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c4"), "n" : 1106, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c5"), "n" : 1107, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c6"), "n" : 1108, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c7"), "n" : 1109, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c8"), "n" : 1110, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c9"), "n" : 1111, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02ca"), "n" : 1112, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cb"), "n" : 1113, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cc"), "n" : 1114, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cd"), "n" : 1115, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02ce"), "n" : 1116, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cf"), "n" : 1117, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02d0"), "n" : 1118, "m" : "test" }
```



__실험: save로 기존 입력된 데이터 변경 시도__: 별개의 데이터로 입력된다

```sql
> db.things.save({empno : 1101, ename : "Blake", dept : "account"})
WriteResult({ "nInserted" : 1 })
> db.things.find()
{ "_id" : ObjectId("600a74ed525efb0c370f02d4"), "empno" : 1101, "ename" : "Blake", "dept" : "account" }
```



__db.emp.update({k:v}, {$set: {k:v}});__

```sql
db.things.update({n:1103}, { $set: {ename : "standford", dept : "research"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.things.update({n:1103}, { $set: {ename : "standford", dept : "research"}})
> db.things.update({n:1104}, { $set: {ename : "John", dept : "inventory"}})
> db.things.update({n:1105}, { $set: {ename : "Jeo", dept : "accounting"}})
> db.things.update({n:1106}, { $set: {ename : "king", dept : "research"}})
> db.things.update({n:1107}, { $set: {ename : "adams", dept : "personel"}})
> db.things.update({n:1108}, { $set: {ename : "blake", dept : "inventory"}})
> db.things.update({n:1109}, { $set: {ename : "smith", dept : "accounting"}})
> db.things.update({n:1110}, { $set: {ename : "allen", dept : "research"}})
> db.things.update({n:1119}, { $set: {ename : "clief", dept : "human resource"}})
> db.things.update({n:1120}, { $set: {ename : "miller", dept : "personel"}})
```



```sql
-- 내용이 추가되었음을 확인할 수 있다

> db.things.find()
{ "_id" : ObjectId("600a6d5a525efb0c370f02bd"), "ename" : "smith" }
{ "_id" : ObjectId("600a6d61525efb0c370f02be"), "empno" : 1101 }
{ "_id" : ObjectId("600a6ff9525efb0c370f02bf"), "empno" : 1102, "ename" : "king" }
{ "_id" : ObjectId("600a70c9525efb0c370f02c0"), "empno" : 1101, "job" : "student" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c1"), "n" : 1103, "m" : "test", "dept" : "research", "ename" : "standford" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c2"), "n" : 1104, "m" : "test", "dept" : "inventory", "ename" : "John" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c3"), "n" : 1105, "m" : "test", "dept" : "accounting", "ename" : "Jeo" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c4"), "n" : 1106, "m" : "test", "dept" : "research", "ename" : "king" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c5"), "n" : 1107, "m" : "test", "dept" : "personel", "ename" : "adams" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c6"), "n" : 1108, "m" : "test", "dept" : "inventory", "ename" : "blake" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c7"), "n" : 1109, "m" : "test", "dept" : "accounting", "ename" : "smith" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c8"), "n" : 1110, "m" : "test", "dept" : "research", "ename" : "allen" }
{ "_id" : ObjectId("600a72d6525efb0c370f02c9"), "n" : 1111, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02ca"), "n" : 1112, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cb"), "n" : 1113, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cc"), "n" : 1114, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cd"), "n" : 1115, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02ce"), "n" : 1116, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02cf"), "n" : 1117, "m" : "test" }
{ "_id" : ObjectId("600a72d6525efb0c370f02d0"), "n" : 1118, "m" : "test" }

```





__db.emp.find().sort({k:v});__



### 데이터 제거

```sql
> db.things.remove({m : "test"})
> db.things.find()
{ "_id" : ObjectId("600a6d5a525efb0c370f02bd"), "ename" : "smith" }
{ "_id" : ObjectId("600a6d61525efb0c370f02be"), "empno" : 1101 }
{ "_id" : ObjectId("600a6ff9525efb0c370f02bf"), "empno" : 1102, "ename" : "king" }
{ "_id" : ObjectId("600a70c9525efb0c370f02c0"), "empno" : 1101, "job" : "student" }
{ "_id" : ObjectId("600a74ed525efb0c370f02d4"), "empno" : 1101, "ename" : "Blake", "dept" : "account" }
```

```sql
> db.things.remove({}) -- remove는 데이터만 지운다
WriteResult({ "nRemoved" : 5 })
> db.things.find()
> show collections --remove를 해도 컬렉션은 아직 남아있다
thing
things
> db.things.drop() --drop하면 없어짐
true
> show collections
thing
> db.thing.drop()
true
> show collections
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```





__db.emp.remove({k:v});__





### SQL과 Mongo Query 비교

| SQL                                                      | Mongo Query                                |
| -------------------------------------------------------- | ------------------------------------------ |
| CREATE TABLE emp<br /><br />(empno Number, ename Number) | db.createCollection("emp")                 |
| INSERT INTO emp VALUES(3,5)                              | db.emp.insert({empno:3, ename:5})          |
| SELECT * FROM emp                                        | db.emp.find()                              |
| SELECT empno, ename FROM emp                             | db.emp.find({}, {empno:1, ename:1})        |
| SELECT * FROM emp WHERE empno = 3                        | db.emp.find({empno:3})                     |
| SELECT empno, ename FROM emp WHERE empno = 3             | db.emp.find({empno:3}, {empno:1, ename:1}) |







