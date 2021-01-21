# RDBMS

### - MariaDB 중심



## 1. 업무분석



## 2. 데이터베이스 생성

__CREATE DATABASE__



## 3. 테이블 생성/삭제

__CREATE TABLE__

```sql
CREATE TABLE indexTBL
	(first_name varchar(14),
	 last_name varchar(16),
	 hire_date date);
```



```sql
USE sqlDB;
CREATE TABLE bigTBL1 (SELECT * FROM employees.employees);
CREATE TABLE bigTBL2 (SELECT * FROM employees.employees);
CREATE TABLE bigTBL3 (SELECT * FROM employees.employees);

DELETE FROM bigTBL1; -- 테이블 남아있음 (영향받은 행 기록 남음)
DROP TABLE bigTBL2; -- 테이블이 사라짐
TRUNCATE TABLE bigTBL3; -- 테이블 남아있음 (영향받은 행 기록 없음)

```





## 4. 데이터 입력(변경)

### 1) INSERT

__INSERT INTO__

```sql
INSERT INTO indexTBL 
	SELECT first_name, last_name, hire_date 
	FROM employees.employees
	LIMIT 500;
```



__id 자동 생성(AUTO_INCREMENT)__

```sql
USE  sqlDB;
CREATE TABLE testTBL2 
  (id  int AUTO_INCREMENT PRIMARY KEY, 
   userName char(3), 
   age int );
INSERT INTO testTBL2 VALUES (NULL, '지민', 25);
INSERT INTO testTBL2 VALUES (NULL, '유나', 22);
INSERT INTO testTBL2 VALUES (NULL, '유경', 21);

SELECT * FROM testTBL2
```



__증가값 지정: AUTO_INCREAMENT_INCREAMENT__

```sql
USE  sqlDB;
CREATE TABLE testTBL3 
  (id  int AUTO_INCREMENT PRIMARY KEY, 
   userName char(3), 
   age int );
   
ALTER TABLE testTBL3 AUTO_INCREMENT=1000;
SET @@auto_increment_increment=3;
INSERT INTO testTBL3 VALUES (NULL, '나연', 20);
INSERT INTO testTBL3 VALUES (NULL, '정연', 18);
INSERT INTO testTBL3 VALUES (NULL, '모모', 19);

SELECT * FROM testTBL3;
```



__대량의 데이터 삽입하기__

```sql
USE sqlDB;
CREATE TABLE testTBL4 (id int, Fname varchar(50), Lname varchar(50));
INSERT INTO testTBL4 
  SELECT emp_no, first_name, last_name -- VALUES 대신 SELECT 절 사용(열 수, 타입 맞춰야 함)
  FROM employees.employees ; -- employees 테이블의 정보를 집어넣음
```

```sql
-- 생성할 때 바로 넣을수도 있다
-- 데이터만 들어갈 뿐, pk 등 제약조건은 들어가지 않는다
USE sqlDB;
CREATE TABLE testTBL5
	(SELECT emp_no, first_name, last_name FROM employees.employees) ;
```



### 2) 데이터의 수정: UPDATE

__update절 뒤에 where 조건절이 붙어야 하는지 항상 주의해서 확인하고 사용하자!__

- 업데이트할 테이블 내용 먼저 확인:  SELECT * FROM
- update (with where)
- 업데이트된 내용 확인

```sql
-- Fname이 Kyoichi인 데이터의 Lname을 모두 '없음'으로 변경
UPDATE testTBL4
	SET Lname = '없음'
	WHERE Fname = 'Kyoichi';
```

```sql
-- price 를 일괄 1.5배
USE sqlDB;
UPDATE buyTBL2 SET price = price * 1.5;
```



### 3) 데이터의 삭제(DELETE FROM)

__신중하게 지우자!__

- 지우고싶은 테이블을 먼저 확인: SELECT * FROM
- DELETE FROM
- 지워졌는지 확인: SELECT * FROM 

```sql
USE sqlDB;
DELETE FROM testTBL4 WHERE Fname = 'Aamer';
DELETE FROM testTBL4 WHERE Fname = 'Mary' LIMIT 5; --ORDER BY 와도 함께 쓸 수 있다
```

 

### 4) 조건부 데이터 입력, 변경: 첫줄에서 에러가 나도 계속 입력해주는 옵션 

__INSERT IGNORE__

__ON DUPLICATE KEY UPDATE__



### 5) WITH / CTE

- 최근 문법
- 중첩에 사용
- 뷰와 용도는 비슷하지만 구문이 끝나면 같이 소멸 (속도가 빠름)



__WITH: 비재귀적 CTE__

```
WITH abc(userid, total)
AS
	(SELECT userid, SUM(price*amount)
	FROM buyTBL GROUPBY userid)
```



## 5. 데이터 조회/활용

```sql
SELECT * FROM indexTBL;
```

### 



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

- MariaDB에서 제공해주는 프로그래밍 기능 ()
- SQL 문을 하나로 묶어서 편리하게 사용하는 기능: 속도가 빠르다
- 직접 작성하는 것보다 가져다 쓰는 경우가 더 많을 것!

```sql
DELIMITER $$
CREATE PROCEDURE myProc()
BEGIN -- 이 부분에 SQL 프로그래밍 코딩
	SELECT * FROM memberTBL WHERE memberName = '당탕이' ;
	SELECT * fROM productTBL WHERE productName = '냉장고' ;
END $$
DELIMITER ;

CALL myProc() ;
```



__IF ELSE__

```sql
DROP PROCEDURE IF EXISTS ifProc; -- 기존에 만든적이 있다면 삭제

DELIMITER $$
CREATE PROCEDURE ifProc()
BEGIN
  DECLARE var1 INT;  -- var1 변수선언
  SET var1 = 100;  -- 변수에 값 대입

  IF var1 = 100 THEN  -- 만약 @var1이 100이라면,
		SELECT '100입니다.';
  ELSE
    SELECT '100이 아닙니다.';
  END IF;
END $$
DELIMITER ;

CALL ifProc() ;
```

```sql
DROP PROCEDURE IF EXISTS ifProc3; 
DELIMITER $$
CREATE PROCEDURE ifProc3()
BEGIN
    DECLARE point INT ;
    DECLARE credit CHAR(1);
    SET point = 77 ;
    
    IF point >= 90 THEN
			SET credit = 'A';
    ELSEIF point >= 80 THEN
			SET credit = 'B';
    ELSEIF point >= 70 THEN
			SET credit = 'C';
    ELSEIF point >= 60 THEN
			SET credit = 'D';
    ELSE
			SET credit = 'F';
    END IF;
    SELECT CONCAT('취득점수==>', point), CONCAT('학점==>', credit);
END $$
DELIMITER ;
CALL ifProc3();
```



__CASE__

```sql
DROP PROCEDURE IF EXISTS caseProc; 
DELIMITER $$
CREATE PROCEDURE caseProc()
BEGIN
    DECLARE point INT ;
    DECLARE credit CHAR(1);
    SET point = 77 ;
    
    CASE 
			WHEN point >= 90 THEN
				SET credit = 'A';
			WHEN point >= 80 THEN
				SET credit = 'B';
			WHEN point >= 70 THEN
				SET credit = 'C';
			WHEN point >= 60 THEN
				SET credit = 'D';
			ELSE
				SET credit = 'F';
    END CASE;
    SELECT CONCAT('취득점수==>', point), CONCAT('학점==>', credit);
END $$
DELIMITER ;

CALL caseProc();
```



__WHILE__

```sql
DROP PROCEDURE IF EXISTS whileProc2; 

DELIMITER $$
CREATE PROCEDURE whileProc2()
BEGIN
    DECLARE i INT; -- 1에서 100까지 증가할 변수
    DECLARE hap INT; -- 더한 값을 누적할 변수
    SET i = 1;
    SET hap = 0;

    myWhile: WHILE (i <= 100) DO  -- While문에 label을 지정
			IF (i%7 = 0) THEN
				SET i = i + 1;     
				ITERATE myWhile; -- 지정한 label문으로 가서 계속 진행
		END IF;
        
       SET hap = hap + i; 
       IF (hap > 1000) THEN 
					LEAVE myWhile; -- 지정한 label문을 떠남. 즉, While 종료.
		END IF;
       SET i = i + 1;
    END WHILE;

    SELECT hap;   
END $$
DELIMITER ;

CALL whileProc2();
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





# SQL 고급

### 변수의 사용

```sql
USE sqlDB;
SET @myVar1 = 5 ;
SET @myVar2 = 3 ;
SET @myVar3 = 4.25 ;
SET @myVar4 = '가수 이름==> ' ;

SELECT @myVar1 ;
SELECT @myVar2 + @myVar3 ;

SELECT @myVar4 , Name FROM userTBL WHERE height > 180 ;
```



### LIMIT 에서 변수 사용하기

```sql
SET @myVar1 = 3 ;
PREPARE myQuery 
    FROM 'SELECT Name, height FROM userTBL ORDER BY height LIMIT ?';
EXECUTE myQuery USING @myVar1 ;
```



### 데이터 형변환

__데이터 형식 변환 함수: CAST, CONVERT__

```sql
SELECT CAST(AVG(amount) AS SIGNED INTEGER) AS '평균 구매 개수' FROM buyTBL ;
-- 또는
SELECT CONVERT(AVG(amount) , SIGNED INTEGER) AS '평균 구매 개수' FROM buyTBL ;
```

```sql
SELECT CAST('2022$12$12' AS DATE);
SELECT CAST('2022/12/12' AS DATE);
SELECT CAST('2022%12%12' AS DATE);
SELECT CAST('2022@12@12' AS DATE);
```



__암시적인 형변환: CONCAT__()

```sql
SELECT num, CONCAT(CAST(price AS CHAR(10)), 'X', CAST(amount AS CHAR(4)) ,'=' )  AS '단가X수량', price*amount AS '구매액' 
FROM buyTBL ;
```

```sql
SELECT '100' + '200' ; -- 문자와 문자를 더함 (정수로 변환돼서 연산됨)
SELECT CONCAT('100', '200'); -- 문자와 문자를 연결 (문자로 처리)
SELECT CONCAT(100, '200'); -- 정수와 문자를 연결 (정수가 문자로 변환돼서 처리)
SELECT 1 > '2mega'; -- 정수 2로 변환되어서 비교
SELECT 3 > '2MEGA'; -- 정수 2로 변환되어서 비교
SELECT 0 = 'mega2'; -- 문자는 0으로 변환됨
```



### 내장함수

- 보통 sql 보다 프로그램에서 데이터를 정제하는 경우가 더 많다

__1) 제어 흐름함수__

__IF__

__IFNULL(수식1, 수식2)__: 수식1이 null인지 판단

__NULLIF(수식1, 수식2)__: 수식1과 수식2가 같은지 판단

__CASE ~ WHEN ~ELSE ~ END__

```sql
SELECT	CASE 10
				WHEN 1 THEN '일'
				WHEN 5 THEN '오'
				WHEN 10 THEN '십' -- '십'이 반환됨
				ELSE '모름'
		END;
```



__문자열 함수__

- ASCII, CHAR

```sql
SELECT ASCII('A'), CHAR(65)
```



- BIT_LENGTH(문자열), CHAR_LENGTH(문자열), LENGTH(문자열)

```sql
SELECT BIT_LENGTH('abc')
```



- ELT

```sql
SELECT ELT(2, '하나', '둘', '셋'), FIELD('둘', '하나', '둘', '셋'), FIND_IN_SET('둘', '하나,둘,셋'), INSTR('하나둘셋', '둘'), LOCATE('둘', '하나둘셋');
```



- INSERT

```sql
SELECT INSERT('abcdefghi', 3, 4, '@@@@'), INSERT('abcdefghi', 3, 2, '@@@@');
SELECT LEFT('abcdefghi', 3), RIGHT('abcdefghi', 3);
SELECT LOWER('abcdEFGH'), UPPER('abcdEFGH');
```



- SUBSTRING(문자열, 시작위치, 길이)

```sql
SELECT SUBSTRING('대한민국만세', 3, 2);
```



- SUBSTRING_INDEX(문자열, 구분자, 횟수)

```sql
SELECT SUBSTRING_INDEX('cafe.naver.com', '.', 2),  SUBSTRING_INDEX('cafe.naver.com', '.', -2);
```



__날짜 및 시간함수__

- SYSDATE(): 현재 연-월-일 시:분:초

```
INSERT INTO 
```



__시스템 정보 함수__

- USER(), DATABASE()

```sql
SELECT USER() ;
SELECT CURRENT_USER() ;
```



__순위함수__

- ROW_COUNT()
- ROW_NUMBER()

```sql
USE sqlDB;
SELECT ROW_NUMBER() OVER(ORDER BY height DESC) "키큰순위", name, addr, heigh
FROM userTBL;
```



### 피벗과  JSON

__피벗의 구현__

``` sql
USE sqlDB;
CREATE TABLE pivotTest
   (  uName CHAR(3),
      season CHAR(2),
      amount INT );

INSERT  INTO  pivotTest VALUES
	('김범수' , '겨울',  10) , ('윤종신' , '여름',  15) , ('김범수' , '가을',  25) , ('김범수' , '봄',    3) ,
	('김범수' , '봄',    37) , ('윤종신' , '겨울',  40) , ('김범수' , '여름',  14) ,('김범수' , '겨울',  22) ,
	('윤종신' , '여름',  64) ;
SELECT * FROM pivotTest;

SELECT uName,
	SUM(IF(season='봄', amount, 0)) AS '봄',
	SUM(IF(season='여름', amount, 0)) AS '여름',
	SUM(IF(season='가을', amount, 0)) AS '가을',
	SUM(IF(season='겨울', amount, 0)) AS '겨울',
	SUM(amount) AS '합계'
FROM pivotTest
GROUP BY uName ;
```







__JSON데이터__

- json 이 많이 사용되기 시작하면서 지원

```

```





## 조인

### INNER JOIN

- 조인이라고 하면 보통 "INNER JOIN"

```sql
USE sqlDB;
SELECT *
FROM buyTBL -- buyTB과 userTBL의 위치를 바꾸면 출력되는 순서가 달라짐
	INNER JOIN userTBL 
		ON buyTBL.userID = userTBL.userID
WHERE buyTBL.userID = 'JYP';
-- 가독성을 위해 첫번째 테이블을 기준으로 WHERE의 조건을 정한다(권고)
```



__alias__

```sql
SELECT U.userID, U.name, B.prodName, U.addr, CONCAT(U.mobile1, U.mobile) AS '연락처'
FROM userTBL U
	INNER JOIN buyTBL B
	ON U.userID = B.userID
WHERE B.userID = U.userID;
```



__DISTINCT__

```sql
SELECT DISTINCE U.userID, U.name, U.addr
FROM userTBL U
	INNER JOIN buyTBL B
	ON U.userID = B.userID
ORDER BY U.userID;
```



__EXISTS: 속도는 조인이 더 빠르다__

```sql
SELECT U.userID, U.name, U.addr
FROM userTBL U
WHERE EXISTS (
	SELECT *
	FROM buyTBL B
	WHERE U.userID = B.userID) ;
```





### OUTER JOIN

- INNER JOIN에 비해서 사용하는 경우는 드물다

```sql
USE sqlDB;
SELECT U.userID, U.name, B.prodName, U.addr, CONCAT(U.mobile1, U.mobile2)  AS '연락처'
   FROM userTBL U
      LEFT OUTER JOIN buyTBL B -- LEFT OUTER JOIN: 왼쪽 테이블의 것은 모두 출력한다
         ON U.userID = B.userID 
   ORDER BY U.userID;

SELECT U.userID, U.name, B.prodName, U.addr, CONCAT(U.mobile1, U.mobile2)  AS '연락처'
   FROM buyTBL B 
      RIGHT OUTER JOIN userTBL U
         ON U.userID = B.userID 
   ORDER BY U.userID;
```



### FULL OUTER JOIN

```sql
SELECT S.stdName, S.addr, C.clubName, C.roomNo
   FROM stdTBL S 
      LEFT OUTER JOIN stdclubTBL SC
          ON S.stdName = SC.stdName
      LEFT OUTER JOIN clubTBL C
          ON SC.clubName = C.clubName
UNION 
SELECT S.stdName, S.addr, C.clubName, C.roomNo
   FROM stdTBL S
      LEFT OUTER JOIN stdclubTBL SC
          ON SC.stdName = S.stdName
      RIGHT OUTER JOIN clubTBL C
          ON SC.clubName = C.clubName;
```





### SELF JOIN

- 대표적인 예: 조직도

```sql
-- 테이블 생성

USE sqlDB;
CREATE TABLE empTbl (emp CHAR(3), manager CHAR(3), empTel VARCHAR(8));

INSERT INTO empTbl VALUES(N'나사장',NULL,'0000');
INSERT INTO empTbl VALUES(N'김재무',N'나사장','2222');
INSERT INTO empTbl VALUES(N'김부장',N'김재무','2222-1');
INSERT INTO empTbl VALUES(N'이부장',N'김재무','2222-2');
INSERT INTO empTbl VALUES(N'우대리',N'이부장','2222-2-1');
INSERT INTO empTbl VALUES(N'지사원',N'이부장','2222-2-2');
INSERT INTO empTbl VALUES(N'이영업',N'나사장','1111');
INSERT INTO empTbl VALUES(N'한과장',N'이영업','1111-1');
INSERT INTO empTbl VALUES(N'최정보',N'나사장','3333');
INSERT INTO empTbl VALUES(N'윤차장',N'최정보','3333-1');
INSERT INTO empTbl VALUES(N'이주임',N'윤차장','3333-1-1');

-- SELF JOIN 활용
-- 우대리 상관의 연락처를 확인
SELECT A.emp AS '부하직원', B.emp AS '직속상관', B.empTel AS '직속상관연락처'
FROM empTbl A
	INNER JOIN empTbl B
	ON A.manager = B.emp
WHERE A.emp = '우대리';
```



### UNION/UNION ALL/NOT IN/IN

__NOT IN__

```sql
-- 전화가 없는 사람을 제외하고 조회
SELECT name, CONCAT(mobile1, mobile2) AS '전화번호'
FROM userTBL
WHERE name NOT IN (
  SELECT name
  FROM userTBL
  WHERE mobile1 IS NULL) ;
```



__IN__

```sql
-- 전화가 없는 사람만 조회
SELECT name, CONCAT(mobile1, mobile2) AS '전화번호'
FROM userTBL
WHERE name IN (
  SELECT name
  FROM userTBL
  WHERE mobile1 IS NULL) ;
```



## SQL 프로그래밍

### 오류처리

```sql
DROP PROCEDURE IF EXISTS errorProc;

DELIMITER $$
CREATE PROCEDURE errorProc()
BEGIN
	DECLARE CONTINUE HANDLER FOR 1146 SELECT '테이블이 없어요ㅠㅠ' AS '메세지';
	SELECT * FROM noTable; --noTable은 없음
END $$
DELIMITER;

CALL errorProc();
```



```sql
DROP PROCEDURE IF EXISTS errorProc2;

DELIMITER $$
CREATE PROCEDURE errorProc2()
BEGIN
	DECLARE CONTINUEW HANDLER FOR SQLEXCEPTION
	BEGIN
			SHOW ERRORS; -- 오류 메세지를 보여준다
			SELECT '오류가 발생했네요. 작업은 취소시켰습니다.' AS '메세지';
			ROLLBACK; --오류 발생시 작업을 롤백시킨다
	END;
	INSERT INTO userTBL VALUES('LSG', '이상구', 1988, '서울', NULL,
					NULL, 170, CURRENT_DATE()); -- 중복되는 아이디어이므로 오류 발생
END $$
DELIMITER ;

CALL errorProc2();
```



### 동적 SQL

- PREPARE: SQL문을 실행하지 않고 미리 준비해놓고 EXECUTE문으로 실행
- 실행후에는 DEALLOCATE PREPARE로 문장 해제

```sql
USE sqlDB;
PREPARE myQuery FROM 'SELECT * FROM userTBL WHERE userID = "EJW"';
EXECUTE myQuery
DEALLOCATE PREPARE myQuery;
```



- PREPARE문에 ?으로 향후 입력될 값을 비워놓고 EXECUTE USING으로 값 전달 가능

```sql
USE sqlDB;
DROP TABLE IF EXISTS myTable;
CREATE TABLE myTable (id INT AUTO_INCREMENT PRIMARY KEY, mDate DATETIME);

SET @curDATE = CURRENT_TIMESTAP(); --현재 날짜와 시간

PREPARE myQuery FROM 'INSERT INTO myTable VALUES(NULL, ?)';
EXECUTE myQuery USING @curDATE;
DEALLOCATE PREPARE myQuery;

SELECT * FROM myTable;
```





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



