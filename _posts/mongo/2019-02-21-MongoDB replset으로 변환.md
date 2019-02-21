---
layout: post
title: MongoDB replset으로 변환  
categories : mongodb
refer1 : https://www.youtube.com/watch?v=9WeCVKFuZVQ
refer2 : https://askubuntu.com/questions/257714/how-to-change-the-location-that-mongodb-uses-to-store-its-data
refer3 : https://stackoverflow.com/questions/5961145/changing-mongodb-data-store-directory
refer4 : https://stackoverflow.com/questions/31657010/how-to-set-default-dbpath-for-mongodb-in-windows-7
refer5 : default configuration 파일 없다 https://stackoverflow.com/questions/14453651/mongo-default-config
---

--- 

<br><br>
## 1. 도입 : 
<br>

 MongoDB 3.6 version 이후 도입된 **Change Stream** 기능은 DB의 변경 사항을 real time으로 관찰할 수 있게 해준다. 즉 프로그램은 Change Stream을 통해서
DB의 변화를 감지하고 그에 따른 Response를 할 수 있다. 단일 Collection부터 Database뿐만 아니라 배포한 모든 것들에 대해서 해당 기능을 적용할 수 있다. 또한 특정 변경 사항에 filter를 적용해 특정 변동에 한정해서 대응할 수 있다. 

<br>
 하지만 Change Stream을 적용하려면 MongoDB를 Replica Set으로 변경해야 한다. 따라서 이번 글에서는 DB를 **Replica Set**으로 변경하는 방법에 대해 다루려고
 한다. 

<br><br>

## 2. Instance Replica Set으로 변경/생성하기 

<br>

1) 실행 중인 Stand alone Instance를 shutdown(프로세스 종료)한다

```
use admin  // admin database에 대하여 shutdown 명령어를 실행해야 한다는 점에 유의해야 한다
db.shutdownServer()
```
<br>

2) `--Replica Set` 옵션을 추가하여 인스턴스를 다시 실행한다.

아래의 명령어를 실행해서 Stand alone instance를 Replica Set의 멤버로 다시 실행한다. 
이 때 dbpath를 파라미터로 전달해주어야 한다. 기존 Stand alone Instance의 경로를 적거나, 새로 인스턴스 파일을 만들 경우 storage로 사용할 새로운 파일의 경로를 적는다. 주의해야 할 점은 별도의 파일 설정을 해주지 않았다면, 현재 인스턴스의 default path는 `'data/db'`라는 점이다. 따라서 현재 인스턴스를 Replica Set으로 변경하고 싶은 경우 path에 `'C:\data\db'` 명시하면 된다. 

```
mongod --port 27017 --dbpath /srv/mongodb/db0 --replSet rs0 
```

<br>

3) Replica Set을 초기화한다. 

```
rs.initiate()
```

<br><br>

## 3. Service로 변경한 Instance 실행하기 

<br>

 Replica Set으로 인스턴스를 변경한 후 프로젝트를 실행하면서 곤혹스러웠던 점은 컴퓨터를 재부팅할 경우 기존 Instance를(내 경우에는 default path의 Instance) Stand alone으로 재실행 한다는 것이었다.

```
mongod --dbpath /srv/mongodb/db0 --replSet rs0 
```
 
 위와 같이 매번 path를 지정해주면서 ReplSet으로 실행시킬 수 있다. 하지만 프로젝트 과정에서 별도의 설정없이 mongodb를 사용했다면 이러한 과정은 매우 불편할 것이다.
따라서 부팅시 원하는 Instance를 실행하기 위해서 **Window Service**를 변경하는 과정이 필요하다. 아래의 과정은 **관리자 권한**으로 cmd를 실행한 후 따라하면 된다.

<br>

1) MongoDB 서비스 종료하기 

```
net stop MongoDB
```
<br>

2) **mongod.cfg** 파일에서 path 수정 (새로 Replica Set 디렉토리를 설정한 경우에만)

mongod.cfg 파일은 MongoDB 폴더의 하위 폴더인 bin 파일에 존재한다. 주의해야 할 점은 mongoDB를 실행할 때 자동으로 config파일의 설정 사항을 반영하지 않는다는 점이다. config 파일의 내용이 아니라 default로 설정되어 있는 설정에 따라 동작하기 때문이다. 따라서 아래의 과정을 통해 mongod.cfg파일의 path를 변경했다 할지라도 다음 번 실행시에 어떠한 변화도 일어나지 않는다. 실행 시 별도의 `--config` 옵션을 통해 사용할 config 파일의 경로를 명시해야 한다.

```
# Where and how to store data.
storage:
  dbPath: C:\data\db1 //이 부분을 원하는 path로 변경한다. 
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  C:\Program Files\MongoDB\Server\4.0\log\mongod.log // log파일의 경로도 별도로 설정해 줄 수 있다.

```
<br>

3) 기존 mongoDB 서비스를 제거한다. 

```
mongod --remove
```
<br>

4) 새로운 Service를 정의한 후 실행한다.

`--config`와 `--replSet` 옵션을 통해 config에 지정된 path의 Instance를 rs0이라는 ReplSet으로 실행하는 서비스를 설치할 수 있다.

```
mongod --config "C:\etc\mongod.cfg" --replSet rs0 --install --serviceName "MongoDB"
net start MongoDB
```


<br><br>

## 4. 추가내용

<br>

위의 과정을 모두 마치면 부팅시 변경 사항을 토대로 새로운 Instance를 실행하게 된다. 기존의 자료를 생성한 디렉토리로 옮기고자 하는 경우 기존 디렉토리에 있는 파일들을 옮겨 넣으면 된다. 

프로젝트에서 노드를 사용하기 때문에 덧붙이자면, ReplSet DB와 node.js를 연결하고자 하는 경우 아래와 같이 URL에 `?replicaSet=rs0(이름)`을 더해주면 된다.

```
new Mongo("mongodb://localhost:27017/demo?replicaSet=rs0");
```

