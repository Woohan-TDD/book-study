# 실습환경 구성하기

## docker 컨테이너 생성

```shell
$ docker-compose up -d # sql 실행까지 다소 시간이 걸릴 수 있음
```

## 데이터베이스 접속

```shell
$ docker exec -it mysql-tuning-practice /bin/bash # 컨테이너에 접속
$ mysql -u root -p # mysql 콘솔 접속
$ root # 패스워드 입력
```

## 데이터베이스 생성 확인

```shell
mysql> show databases;
mysql> use tuning;
mysql> show tables;
mysql> select count(1) from 사원; # 결과: 300024
```

## docker 컨테이너 종료

```shell
$ docker stop mysql-tuning-practice
```

## docker 컨테이너 재실행

```shell
$ docker start mysql-tuning-practice
```

## docker 컨테이너 제거

```shell
$ docker rm mysql-tuning-practice
```

## 참고 링크

- [실습 github](https://github.com/7ieon/SQLtune)
