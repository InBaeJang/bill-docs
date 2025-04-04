## MySQL 서버 시작

- $ mysql.server start
- $ mysql.server stop
- $ mysql --version // 버전확인
  mysql> status; // 상태 보기
  mysql> SHOW VARIABLES LIKE "%version%"; // 버전 확인

## MySQL 접속 In Mac OS, Linux

- $ mysql -u [username] -p [dbname]
  $ mysql -uroot -p
  $ mysql -u root -p // 이것도 가능
  password: dist7777

## 데이터베이스 기본 명령어

- mysql> SHOW DATABASES;
- mysql> USE dbname;
- mysql> DROP DATABASE [IF EXISTS] dbname;
- mysql> SHOW TABLES;
- mysql> EXPLAIN tablesname;

# mysql> DESCRIBE tablename;

- # mysql> DROP TABLE [IF EXISTS] tablename;

[Installation On Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)

# mysql> use mysql;

- user의 설정된 비밀번호 확인

# mysql> select user, host, authentication_string from user;

- [비밀번호 변경](https://inma.tistory.com/98)

# mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'userPW';

- 변경사항 적용

# mysql> flush privileges;
