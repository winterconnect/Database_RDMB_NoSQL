# RDBMS: MariaDB

## 1. 업무분석



## 2. 데이터베이스 생성

__CREATE DATABASE__



## 3. 테이블 생성

__CREATE TABLE__

```sql
CREATE TABLE indexTBL
	(first_name varchar(14),
	 last_name varchar(16),
	 hire_date date);
```



## 4. 데이터 입력

__INSERT INTO__

```sql
INSERT INTO indexTBL 
	SELECT first_name, last_name, hire_date 
	FROM employees.employees
	LIMIT 500;
```



## 5. 데이터 조회/활용

```sql
SELECT * FROM indexTBL;
```



## 6. 인덱스

__CREATE INDEX__

```sql
CREATE INDEX idx_indexTBL_firstname ON indexTBL(first_name);
```

```sql
EXPLAIN SELECT * FROM indexTBL WHERE first_name = 'Mary';
```





## 7. 뷰

```sql
CREATE VIEW uv_memberTBL
AS
	SELECT memberName, memberAddress
	FROM memberTBL;
```



## 8. 스토어드 프로시저

- MariaDB에서 제공해주는 프로그래밍 기능
- SQL 문을 하나로 묶어서 편리하게 사용하는 기능

```sql
DELIMITER //
CREATE PROCEDURE myProc()
BEGIN
	SELECT * FROM memberTBL WHERE memberName = '당탕이' ;
	SELECT * fROM productTBL WHERE productName = '냉장고' ;
END //
DELIMITER ;
```



## 9. 트리거

```sql
CREATE TABLE deleteMmeberTBL (
	memberID char(8),
	memberName char(5),
	memberAddress char(20),
	deletedDate date
	);
	
DELIMITER //
CREATE TRIGGER trg_deletedMemberTBL -- 트리거 이름
	AFTER DELETE -- 삭제 후에 작동하게 지정
	ON memberTBL -- 트리거를 부착할 테이블
	FOR EACH ROW -- 각 행마다 적용시킴
BEGIN
	-- OLD 테이블의 내용을 백업 테이블에 삽입
	INSERT INTO deletedMemberTBL VALUES 
	(OLD.memberID, OLD.memberName, OLD.memberAddress, CURDATE());
END //
DELIMITER ;
```



## 10. 커서



## 11. 백업/복원





# 데이터베이스 모델링



## 1. 프로젝트의 진행 단계



모델링 실습



게시판을 활용하여 회원들이 글을 쓰고, 댓글을 사용하고, 파일을 업로드할 수 있는 데이터베이스 모델링



객체: 사용자, 게시판, 작성글, 댓글, 파일

1. 사용자: 사용자id(pk), 패스워드, 이름, 가입일시, 성별, 연령, 작성글수, 댓글수 등 (이외 필요한 개인정보)



2. 게시판: 게시판명(pk), 게시글수 등



3. 작성글: 작성글번호(pk), 게시판명(fk), 글 제목, 작성자id(fk), 작성일시, 내용, 조회수, 댓글수, 댓글id



4. 댓글: 댓글id(pk), 작성글번호(fk), 작성자id(fk), 작성일시, 내용, 



5. 파일: 파일id(pk), 파일명, 작성글번호(fk), 업로드일시, 파일형식, 용량, 파일저장위치





1. 업무파악

- 게시판 등록: 제목, 등록내용, 누가(회원), 언제(작성일시), 조회수
- 회원등록: 회원id, 회원pw, 회원이름, 성별, 이메일, 전화번호, 주소 등
- 댓글등록: 회원id(작성자), 댓글내용, 작성일시, 비밀번호 등
- 파일등록: 등록자, 파일명, 파일저장위치, 등록일시, 비밀번호 등



2. 데이터베이스 생성



3. 테이블작성

- UserTBL: UserID(PK), UserPW, UserName, ...
- BoardTBL: Title, Contents, UserID, Date
- ReplyTBL: UserID, Contents, ...
- FileTBL



