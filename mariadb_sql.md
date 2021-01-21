# RDBMS

### - MariaDB 중심



## 개요

> 1. 업무분석: 데이터베이스 모델링(개념 - 논리 - 물리)
> 2. 데이터베이스 생성
> 3. 테이블 생성: 필드명, 데이터타입, 길이, 제약조건
> 4. 데이터 입력/활용
> 5. 인덱스, 뷰, 스토어드 프로시저, 함수, 트리거, 커서
> 6. 백업/복원





## 1. 업무분석

### 1) 데이터베이스 모델링

- 개념 - 논리 - 물리적 모델링





## 2. 데이터베이스 생성

__CREATE DATABASE__

```sql
DROP DATABASE IF EXISTS ShopDB;
DROP DATABASE IF EXISTS ModelDB;
DROP DATABASE IF EXISTS sqlDB;
DROP DATABASE IF EXISTS tableDB;

CREATE DATABASE tableDB;
```





## 3. 테이블 생성/삭제

### 1) 테이블 생성

- 필드명, 데이터타입, 길이, 제약조건 등



__CREATE TABLE__

```sql
CREATE TABLE indexTBL
	(first_name varchar(14),
	 last_name varchar(16),
	 hire_date date);
```



__데이터를 끌어와 테이블 생성__

```sql
USE sqlDB;
CREATE TABLE bigTBL1 (SELECT * FROM employees.employees);
CREATE TABLE bigTBL2 (SELECT * FROM employees.employees);
CREATE TABLE bigTBL3 (SELECT * FROM employees.employees);

DELETE FROM bigTBL1; -- 테이블 남아있음 (영향받은 행 기록 남음)
DROP TABLE bigTBL2; -- 테이블이 사라짐
TRUNCATE TABLE bigTBL3; -- 테이블 남아있음 (영향받은 행 기록 없음)

```



__테이블 생성__

```sql
DROP DATABASE IF EXISTS tableDB;
CREATE DATABASE tableDB;

USE tableDB;
DROP TABLE IF EXISTS buyTBL, userTBL;

CREATE TABLE userTBL -- 회원 테이블
( userID  char(8), -- 사용자 아이디
  name    nvarchar(10), -- 이름
  birthYear   int,  -- 출생연도
  addr	  nchar(2), -- 지역(경기,서울,경남 등으로 글자만 입력)
  mobile1	char(3), -- 휴대폰의국번(011, 016, 017, 018, 019, 010 등)
  mobile2   char(8), -- 휴대폰의 나머지 전화번호(하이픈 제외)
  height    smallint,  -- 키
  mDate    date  -- 회원 가입일
);

CREATE TABLE buyTBL -- 구매 테이블
(  num int, -- 순번(PK)
   userid  char(8),-- 아이디(FK)
prodName nchar(6), -- 물품명
   groupName nchar(4) , -- 분류
   price     int , -- 단가
   amount    smallint -- 수량
);
```



### 3-1 제약조건

- 데이터의 무결성을 지키기 위한 제한된 조건
- 6가지 제약조건
  - PRIMARY KEY: UNIQUE+NOT NULL, 자동으로 클러스터형 인덱스 생성, 하나 이상의 열로 설정 가능
    - 클러스터형 인덱스: 시스템이 만들어주는 인덱스
    - 보통 하나의 열로 지정하지, 두개를 묶어서 지정하는 경우는 거의 없다
  - FOREIGN KEY
  - UNIQUE: NULL 허용, Alternate Key 라고도 부름
  - CHECK: 입력되는 데이터를 점검하는 기능 ex) 키에 마이너스값 불가
  - DEFAULT
  - NULL(NOT NULL)



__PRIMARY KEY__

```sql
-- PRIMARY KEY를 무엇으로 할지 가장 먼저 생각하고 정하자
DROP TABLE IF EXISTS buyTBL, userTBL;
CREATE TABLE userTBL 
( userID  char(8) NOT NULL PRIMARY KEY, 
   name    varchar(10) NOT NULL, 
   birthYear   int NOT NULL,  
   addr	  char(2) NOT NULL,
   mobile1	char(3) NULL, 
   mobile2   char(8) NULL, 
   height    smallint NULL,
   mDate    date NULL 
   
DROP TABLE IF EXISTS buyTBL;
CREATE TABLE buyTBL 
(  num int AUTO_INCREMENT NOT NULL PRIMARY KEY , 
   userid  char(8) NOT NULL ,
   prodName char(6) NOT NULL,
   groupName char(4) NULL , 
   price     int  NOT NULL,
   amount    smallint  NOT NULL,
 	 FOREIGN KEY(userid) REFERENCES userTBL(userID) -- FK는 가장 뒤에 적는것이 관례
```



__CONSTRAINT__

```sql
-- 뒤에 constraint 를 붙여서 이름을 지정해줄 수도 있다 예) PK_userTBL_userID
DROP TABLE IF EXISTS userTBL;
CREATE TABLE userTBL 
( userID  CHAR(8) NOT NULL, 
  name    VARCHAR(10) NOT NULL, 
  birthYear   INT NOT NULL,  
  CONSTRAINT PRIMARY KEY PK_userTBL_userID (userID)
```



```sql
-- fk 이름 지정
DROP TABLE IF EXISTS buyTBL;
CREATE TABLE buyTBL 
(  num INT AUTO_INCREMENT NOT NULL PRIMARY KEY , 
   userID  CHAR(8) NOT NULL, 
   prodName CHAR(6) NOT NULL,
   CONSTRAINT FK_userTBL_buyTBL FOREIGN KEY(userID) REFERENCES userTBL(userID)
);
```



- 제약조건은 보통 CREATE TABLE 시에 생성하지만, ALTER TABLE에서도 사용 가능 
  - 나중에 제약이 필요한 경우

```sql
-- 후에 pk 설정
DROP TABLE IF EXISTS userTBL;
CREATE TABLE userTBL 
(   userID  CHAR(8) NOT NULL, 
    name    VARCHAR(10) NOT NULL, 
    birthYear   INT NOT NULL
);
ALTER TABLE userTBL
	ADD CONSTRAINT PK_userTBL_userID PRIMARY KEY (userID);
```



```sql
-- 후에 fk 설정
DROP TABLE IF EXISTS buyTBL;
CREATE TABLE buyTBL 
(  num INT AUTO_INCREMENT NOT NULL PRIMARY KEY,
   userID  CHAR(8) NOT NULL, 
   prodName CHAR(6) NOT NULL 
);
ALTER TABLE buyTBL
    ADD CONSTRAINT FK_userTBL_buyTBL 
    FOREIGN KEY (userID) 
    REFERENCES userTBL(userID);
    
SHOW INDEX FROM buyTBL ;
```



__외래키 제약조건__

- ON UPDATE CASCADE: update 내용이 같이 적용됨
- ON DELETE CASCADE: delete 내용이 같이 적용됨
- 지정하지 않으면 ON UPDATE NO ACTION, ON DELETE NO ACTION



__CHECK__

```sql
-- 출생연도가 1900년 이후 그리고 2020년 이전, 이름은 반드시 넣어야 함.
DROP TABLE IF EXISTS userTBL;
CREATE TABLE userTBL 
( userID  CHAR(8) PRIMARY KEY,
  name    VARCHAR(10), 
  birthYear  INT CHECK (birthYear >= 1900 AND birthYear <= 2020),
  mobile1	char(3) NULL, 
  CONSTRAINT CK_name CHECK (name IS NOT NULL)  -- check 의 이름 지정
);
```



```sql
ALTER TABLE userTbl
	ADD CONSTRAINT CK_mobile1
	CHECK  (mobile1 IN ('010','011','016','017','018','019')) ;
```



```sql
SET foreign_key_checks = 0;  -- 외래 키 제약조건 비활성화
INSERT INTO buyTBL VALUES(NULL, 'BBK', N'모니터', N'전자', 200,  5);
INSERT INTO buyTBL VALUES(NULL, 'KBS', N'청바지', N'의류', 50,   3);
INSERT INTO buyTBL VALUES(NULL, 'BBK', N'메모리', N'전자', 80,  10);
INSERT INTO buyTBL VALUES(NULL, 'SSK', N'책'    , N'서적', 15,   5);
INSERT INTO buyTBL VALUES(NULL, 'EJW', N'책'    , N'서적', 15,   2);
INSERT INTO buyTBL VALUES(NULL, 'EJW', N'청바지', N'의류', 50,   1);
INSERT INTO buyTBL VALUES(NULL, 'BBK', N'운동화', NULL   , 30,   2);
INSERT INTO buyTBL VALUES(NULL, 'EJW', N'책'    , N'서적', 15,   1);
INSERT INTO buyTBL VALUES(NULL, 'BBK', N'운동화', NULL   , 30,   2);
SET foreign_key_checks = 1; -- 외래 키 제약조건 활성화
```



__DEFAULT__

```sql
DROP TABLE IF EXISTS userTBL;
CREATE TABLE userTBL 
( userID  	char(8) NOT NULL PRIMARY KEY,  
  name    	varchar(10) NOT NULL, 
  birthYear	int NOT NULL DEFAULT -1,
  addr	  	char(2) NOT NULL DEFAULT '서울', -- 값이 없으면 '서울'이 default로 들어감
  mobile1	char(3) NULL, 
  mobile2	char(8) NULL, 
  height	smallint NULL DEFAULT 170, 
  mDate    	date NULL
);
```



```sql
-- ALTER 문에서 DEFAULT 지정
DROP TABLE IF EXISTS userTBL;
CREATE TABLE userTBL 
( userID	char(8) NOT NULL PRIMARY KEY,  
  name		varchar(10) NOT NULL, 
  birthYear	int NOT NULL ,
  addr		char(2) NOT NULL,
  mobile1	char(3) NULL, 
  mobile2	char(8) NULL, 
  height	smallint NULL, 
  mDate	date NULL 
);
ALTER TABLE userTBL
	ALTER COLUMN birthYear SET DEFAULT -1;
ALTER TABLE userTBL
	ALTER COLUMN addr SET DEFAULT '서울';
ALTER TABLE userTBL
	ALTER COLUMN height SET DEFAULT 170;
	
-- default 문은 DEFAULT로 설정된 값을 자동 입력한다.
INSERT INTO userTBL VALUES ('LHL', '이혜리', default, default, '011', '1234567', default, '2022.12.12');
-- 열이름이 명시되지 않으면 DEFAULT로 설정된 값을 자동 입력한다
INSERT INTO userTBL(userID, name) VALUES('KAY', '김아영');
-- 값이 직접 명기되면 DEFAULT로 설정된 값은 무시된다.
INSERT INTO userTBL VALUES ('WB', '원빈', 1982, '대전', '019', '9876543', 176, '2023.5.5');

SELECT * FROM userTBL;
```



### 테이블 압축

### 임시 테이블

- 세션 내에서만 존재하고 세션이 닫히면 자동 삭제

```sql
USE employees;
CREATE TEMPORARY TABLE  IF NOT EXISTS  tempTBL (id INT, name CHAR(7));
CREATE TEMPORARY TABLE  IF NOT EXISTS employees (id INT, name CHAR(7));
DESCRIBE tempTBL;
DESCRIBE employees;
```



### 2) 테이블 삭제: DROP TABLE

- 외래키 재약조건의 기준 테이블 삭제 불가
- 외래키가 생성된 외래키 테이블을 먼저 삭제



### 3) 테이블 수정: ALTER TABLE





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

- 자동으로 증가함(default: 1부터 1씩)

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

- 목적: 빠른 검색
- 장단점: 대용량의 테이블에서 검색속도 빨라짐(+), 추가적인 공간 10% 정도 필요(-), 처음 생성시간 오래걸림(-)
- SELECT 가 많은 경우에 유용. INSERT, UPDATE, DELETE 가 많은 경우 굳이 생성할 필요 없음



### 1) 인덱스의 종류

​	(1) 클러스터형 인덱스: 인덱스 생성시 데이터 페이지 전체 다시 정렬

​	(2) 보조 인덱스: 검색속도는 느리지만 데이터 입력/수정/삭제는 덜 느림

### 2) 제약조건과 관련되는 인덱스

### 3) 인덱스의 내부작동

### 4) 인덱스의 생성/변경/삭제

__CREATE INDEX__

```sql
CREATE INDEX idx_userTBL_addr 
   ON userTBL (addr);

SHOW INDEX FROM userTBL ;

SHOW TABLE STATUS LIKE 'userTBL -- 테이블 상태 보기
```

```sql
CREATE INDEX idx_indexTBL_firstname ON indexTBL(first_name);
```

```sql
EXPLAIN SELECT * FROM indexTBL WHERE first_name = 'Mary';
```



__DROP INDEX__

```sql
DROP INDEX idx_userTBL_addr ON userTBL;
```



### 5) 인덱스의 성능

### 6) 인덱스를 생성해야 하는 경우





## 7. 뷰

- 일반 사용자 입장에서는 테이블과 동일하게 사용하는 개체
- 효율성과 보안!
- 보통 읽기 전용으로 사용
- 뷰를 통해 원래 테이블의 데이터를 수정하는 것도 가능(UPDATE 사용)



__뷰 생성__

```sql
CREATE VIEW uv_memberTBL
AS
	SELECT memberName, memberAddress
	FROM memberTBL;
```



__조인 테이블을 뷰로 만들기__

```sql
USE tableDB;
CREATE VIEW v_userTBL
AS
	SELECT userid, name, addr FROM userTBL;

SELECT * FROM v_userTBL;  -- 뷰를 테이블이라고 생각해도 무방

SELECT U.userid, U.name, B.prodName, U.addr, CONCAT(U.mobile1, U.mobile2)  AS '연락처'
FROM userTBL U
  INNER JOIN buyTBL B
     ON U.userid = B.userid ;

-- 조인한 내용을 테이블로 만들어두었기 때문에 계속 사용가능(효율적)
CREATE VIEW v_userbuyTBL
AS
SELECT U.userid, U.name, B.prodName, U.addr, CONCAT(U.mobile1, U.mobile2)  AS '연락처'
FROM userTBL U
	INNER JOIN buyTBL B
	 ON U.userid = B.userid ;

SELECT * FROM v_userbuyTBL WHERE name = N'김범수';
```





## 8. 스토어드 프로그램

## 8-1. 스토어드 프로시저

- MariaDB에서 제공해주는 프로그래밍 기능 ()
- SQL 문을 하나로 묶어서 편리하게 사용하는 기능: 속도가 빠르다
- 직접 작성하는 것보다 가져다 쓰는 경우가 더 많을 것!



__기본문법__

```sql
DELIMITER $$
CREATE PROCEDURE
BEGIN


END $$
DELIMITER;
CALL
```



__프로시저 생성__

```sql
DELIMITER $$
CREATE PROCEDURE myProc()
BEGIN -- 이 부분에 SQL 프로그래밍 코딩
	SELECT * FROM memberTBL WHERE memberName = '당탕이' ;
	SELECT * fROM productTBL WHERE productName = '냉장고' ;
END $$
DELIMITER ; -- 끝이다 라는 것을 알려주는 용도

CALL myProc() ;
```



```sql
USE sqlDB;
DROP PROCEDURE IF EXISTS userProc1;
DELIMITER $$
CREATE PROCEDURE userProc1(IN userName VARCHAR(10))
BEGIN
  SELECT * FROM userTBL WHERE name = userName; 
END $$
DELIMITER ;

CALL userProc1('조관우');
```



```sql
-- 두개의 입력 매개변수가 있는 경우
DROP PROCEDURE IF EXISTS userProc2;
DELIMITER $$
CREATE PROCEDURE userProc2(
    IN userBirth INT, 
    IN userHeight INT
)
BEGIN
  SELECT * FROM userTBL 
    WHERE birthYear > userBirth AND height > userHeight;
END $$
DELIMITER ;

CALL userProc2(1970, 178);
```



```sql
-- OUT 매개변수: 직접 값을 입력하는 게 아니라 값을 받아와서 입력
DROP PROCEDURE IF EXISTS userProc3;
DELIMITER $$
CREATE PROCEDURE userProc3(
    IN txtValue CHAR(10),
    OUT outValue INT
)
BEGIN
  INSERT INTO testTBL VALUES(NULL,txtValue);
  SELECT MAX(id) INTO outValue FROM testTBL; 
END $$
DELIMITER ;

CREATE TABLE IF NOT EXISTS testTBL(
    id INT AUTO_INCREMENT PRIMARY KEY, 
    txt CHAR(10)
);

CALL userProc3 ('테스트값', @myValue);
SELECT CONCAT('현재 입력된 ID 값 ==>', @myValue);
```



__IFELSE PROCEDURE__

```sql
DROP PROCEDURE IF EXISTS ifelseProc;
DELIMITER $$
CREATE PROCEDURE ifelseProc(
    IN userName VARCHAR(10)
)
BEGIN
    DECLARE bYear INT; -- 변수 선언
    SELECT birthYear into bYear FROM userTBL
        WHERE name = userName;
    IF (bYear >= 1980) THEN
            SELECT '아직 젊군요..';
    ELSE
            SELECT '나이가 지긋하네요..';
    END IF;
END $$
DELIMITER ;

CALL ifelseProc ('조용필');
```



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



__CASE PROCEDURE__

```sql
DROP PROCEDURE IF EXISTS caseProc;
DELIMITER $$
CREATE PROCEDURE caseProc(
    IN userName VARCHAR(10)
)
BEGIN
    DECLARE bYear INT; 
    DECLARE tti  CHAR(3);-- 띠
    SELECT birthYear INTO bYear FROM userTBL
        WHERE name = userName;
    CASE 
        WHEN ( bYear%12 = 0) THEN    SET tti = '원숭이';
        WHEN ( bYear%12 = 1) THEN    SET tti = '닭';
        WHEN ( bYear%12 = 2) THEN    SET tti = '개';
        WHEN ( bYear%12 = 3) THEN    SET tti = '돼지';
        WHEN ( bYear%12 = 4) THEN    SET tti = '쥐';
        WHEN ( bYear%12 = 5) THEN    SET tti = '소';
        WHEN ( bYear%12 = 6) THEN    SET tti = '호랑이';
        WHEN ( bYear%12 = 7) THEN    SET tti = '토끼';
        WHEN ( bYear%12 = 8) THEN    SET tti = '용';
        WHEN ( bYear%12 = 9) THEN    SET tti = '뱀';
        WHEN ( bYear%12 = 10) THEN    SET tti = '말';
        ELSE SET tti = '양';
    END CASE;
    SELECT CONCAT(userName, '의 띠 ==>', tti);
END $$
DELIMITER ;

CALL caseProc ('김범수');
```



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



__WHILE PROCEDURE (for문은 없음)__

```sql
DROP TABLE IF EXISTS guguTBL;
CREATE TABLE guguTBL (txt VARCHAR(100)); -- 구구단 저장용 테이블

DROP PROCEDURE IF EXISTS whileProc;
DELIMITER $$
CREATE PROCEDURE whileProc()
BEGIN
    DECLARE str VARCHAR(100); -- 각 단을 문자열로 저장
    DECLARE i INT; -- 구구단 앞자리
    DECLARE k INT; -- 구구단 뒷자리
    SET i = 2; -- 2단부터 계산
    
    WHILE (i < 10) DO  -- 바깥 반복문. 2단~9단까지.
        SET str = ''; -- 각 단의 결과를 저장할 문자열 초기화
        SET k = 1; -- 구구단 뒷자리는 항상 1부터 9까지.
        WHILE (k < 10) DO
            SET str = CONCAT(str, '  ', i, 'x', k, '=', i*k); -- 문자열 만들기
            SET k = k + 1; -- 뒷자리 증가
        END WHILE;
        SET i = i + 1; -- 앞자리 증가
        INSERT INTO guguTBL VALUES(str); -- 각 단의 결과를 테이블에 입력.
    END WHILE;
END $$
DELIMITER ;

CALL whileProc();
SELECT * FROM guguTBL;
```



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



__프로시저 수정과 삭제: ALTER PROCEDURE, DROP PROCEDURE__





## 8-2. 스토어드 함수

__기본문법__

```sql
DELIMITER $$
CREATE FUNCTION
	RETURNS
BEGIN
	RETURN
END $$
DELIMITER;

SELECT 함수이름();
```



```sql
USE sqlDB;
DROP FUNCTION IF EXISTS getAgeFunc;
DELIMITER $$
CREATE FUNCTION getAgeFunc(bYear INT)
    RETURNS INT
BEGIN
    DECLARE age INT;
    SET age = YEAR(CURDATE()) - bYear;
    RETURN age;
END $$
DELIMITER ;

SELECT getAgeFunc(1979);
```



__함수 활용__

```sql
-- 나이차 출력하기
SELECT getAgeFunc(1979) INTO @age1979; --변수에 받음
SELECT getAgeFunc(1997) INTO @age1997;
SELECT CONCAT('1997년과 1979년의 나이차 ==> ', (@age1979-@age1997));

-- 만 나이 출력하기
SELECT userID, name, getAgeFunc(birthYear) AS '만 나이' FROM userTBL;
```



현재 저장된 스토어드 함수의 이름 및 내용 확인: __SHOW CREATE FUNCTION__

```sql
SHOW CREATE FUNCTION getAgeFunc ;
```



__함수삭제__

```sql
DROP FUNCTION getAgeFunc;
```





## 8-3. 커서

- SELECT로 요청한 데이터를 받아놓을 "그릇"
- 커서는 pointer로 데이터를 가리키고 있고,
- 커서의 내용을 가져오는 것이 __"fetch"__
- 포인터를 돌리면서 커서에 있는 데이터를 하나씩 fetch해오는 것



### 처리순서

- 커서의 선언
- 커서 열기
- 커서에서 데이터 가져오기(FETCH)
- 데이터 처리
- 커서 닫기



__커서의 활용__

```sql
DROP PROCEDURE IF EXISTS cursorProc;
DELIMITER $$
CREATE PROCEDURE cursorProc()
BEGIN
    DECLARE userHeight INT; -- 고객의 키
    DECLARE cnt INT DEFAULT 0; -- 고객의 인원 수(=읽은 행의 수)
    DECLARE totalHeight INT DEFAULT 0; -- 키의 합계

    DECLARE endOfRow BOOLEAN DEFAULT FALSE; -- 행의 끝 여부(기본을 FALSE)

    DECLARE userCuror CURSOR FOR-- 커서 선언
        SELECT height FROM userTBL;

    DECLARE CONTINUE HANDLER -- 에러처리: 행의 끝이면 endOfRow 변수에 TRUE를 대입 
        FOR NOT FOUND SET endOfRow = TRUE;

    OPEN userCuror;  -- 커서 열기 (커서가 실행됨)

    cursor_loop: LOOP
        FETCH  userCuror INTO userHeight; -- 고객 키 1개를 대입

        IF endOfRow THEN -- 더이상 읽을 행이 없으면 Loop를 종료
            LEAVE cursor_loop;
        END IF;

        SET cnt = cnt + 1;
        SET totalHeight = totalHeight + userHeight;        
    END LOOP cursor_loop;
    
    -- 고객 키의 평균을 출력한다.
    SELECT CONCAT('고객 키의 평균 ==> ', (totalHeight/cnt));

    CLOSE userCuror;  -- 커서 닫기
END $$
DELIMITER ;

CALL cursorProc();
```



__유령고객 판단__

```sql
USE sqlDB;
ALTER TABLE userTBL ADD grade VARCHAR(5);  -- 고객 등급 열 추가


DROP PROCEDURE IF EXISTS gradeProc;
DELIMITER $$
CREATE PROCEDURE gradeProc()
BEGIN
    DECLARE id VARCHAR(10); -- 사용자 아이디를 저장할 변수
    DECLARE hap BIGINT; -- 총 구매액을 저장할 변수
    DECLARE userGrade CHAR(5); -- 고객 등급 변수
    
    DECLARE endOfRow BOOLEAN DEFAULT FALSE; 

    DECLARE userCuror CURSOR FOR-- 커서 선언
        SELECT U.userid, sum(price*amount)
            FROM buyTBL B
                RIGHT OUTER JOIN userTBL U -- 유령고객을 판단하기 위해 right outer join
                ON B.userid = U.userid
            GROUP BY U.userid, U.name ;

    DECLARE CONTINUE HANDLER 
        FOR NOT FOUND SET endOfRow = TRUE;

    OPEN userCuror;  -- 커서 열기
    grade_loop: LOOP
        FETCH  userCuror INTO id, hap; -- 첫 행 값을 대입
        IF endOfRow THEN
            LEAVE grade_loop;
        END IF;

        CASE  
            WHEN (hap >= 1500) THEN SET userGrade = '최우수고객';
            WHEN (hap  >= 1000) THEN SET userGrade ='우수고객';
            WHEN (hap >= 1) THEN SET userGrade ='일반고객';
            ELSE SET userGrade ='유령고객';
         END CASE;

        UPDATE userTBL SET grade = userGrade WHERE userID = id; -- 새로만든 컬럼에 값을 업데이트
    END LOOP grade_loop;

    CLOSE userCuror;  -- 커서 닫기
END $$
DELIMITER ;

CALL gradeProc();
SELECT * FROM userTBL;
```



__참고) CREATE TABLE 시에 커서와 조건문을 활용하여 데이터를 바로 입력하는 방법도 있다__





## 8-4. 트리거

- 테이블에 삽입, 수정, 삭제 등의 작업(이벤트)이 발생할 때 자동으로 작동되는 개체



__AFTER TRIGGER__

- 작업이 일어났을 때 작동하는 트리거
- __TRUNCATE TABLE__은 트리거 작동하지 않음

```sql
CREATE TABLE deleteMemberTBL (
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



```sql
USE sqlDB;
CREATE TABLE IF NOT EXISTS testTbl (id INT, txt VARCHAR(10));
INSERT INTO testTbl VALUES(1, '이엑스아이디');
INSERT INTO testTbl VALUES(2, '애프터스쿨');
INSERT INTO testTbl VALUES(3, '에이오에이');

DROP TRIGGER IF EXISTS testTrg;
DELIMITER // 
CREATE TRIGGER testTrg  -- 트리거 이름
    AFTER  DELETE -- 삭제후에 작동하도록 지정
    ON testTbl -- 트리거를 부착할 테이블
    FOR EACH ROW -- 각 행마다 적용시킴
BEGIN
	SET @msg = '가수 그룹이 삭제됨' ; -- 트리거 실행시 작동되는 코드들
END // 
DELIMITER ;

SET @msg = '';
INSERT INTO testTbl VALUES(4, '나인뮤지스');
SELECT @msg;
UPDATE testTbl SET txt = '에이핑크' WHERE id = 3;
SELECT @msg;
DELETE FROM testTbl WHERE id = 4;
SELECT @msg;
```



__수정, 삭제된 테이블 내용을 트리거로 확인하기__

```sql
USE sqlDB;
DROP TABLE buyTBL; -- 구매테이블은 실습에 필요없으므로 삭제.
CREATE TABLE backup_userTBL
( userID  char(8) NOT NULL PRIMARY KEY, 
  name    varchar(10) NOT NULL, 
  birthYear   int NOT NULL,  
  addr	  char(2) NOT NULL, 
  mobile1	char(3), 
  mobile2   char(8), 
  height    smallint,  
  mDate    date,
  modType  char(2), -- 변경된 타입. '수정' 또는 '삭제'
  modDate  date, -- 변경된 날짜
  modUser  varchar(256) -- 변경한 사용자
);

DROP TRIGGER IF EXISTS backUserTbl_UpdateTrg;
DELIMITER // 
CREATE TRIGGER backUserTbl_UpdateTrg  -- 트리거 이름
    AFTER UPDATE -- 변경 후에 작동하도록 지정
    ON userTBL -- 트리거를 부착할 테이블
    FOR EACH ROW 
BEGIN
    INSERT INTO backup_userTBL VALUES( OLD.userID, OLD.name, OLD.birthYear, 
        OLD.addr, OLD.mobile1, OLD.mobile2, OLD.height, OLD.mDate, 
        '수정', CURDATE(), CURRENT_USER() );
END // 
DELIMITER ;


DROP TRIGGER IF EXISTS backUserTbl_DeleteTrg;
DELIMITER // 
CREATE TRIGGER backUserTbl_DeleteTrg  -- 트리거 이름
    AFTER DELETE -- 삭제 후에 작동하도록 지정
    ON userTBL -- 트리거를 부착할 테이블
    FOR EACH ROW 
BEGIN
    INSERT INTO backup_userTBL VALUES( OLD.userID, OLD.name, OLD.birthYear, 
        OLD.addr, OLD.mobile1, OLD.mobile2, OLD.height, OLD.mDate, 
        '삭제', CURDATE(), CURRENT_USER() );
END // 
DELIMITER ;
```





__BEFORE TRIGGER__

- 이벤트가 발생하기 전에 작동하는 트리거

- commit을 하기 전에 수행되었던 insert, update, delete 이벤트로 작동

```sql
USE sqlDB;
DROP TRIGGER IF EXISTS userTBL_BeforeInsertTrg;
DELIMITER // 
CREATE TRIGGER userTBL_BeforeInsertTrg  -- 트리거 이름
    BEFORE INSERT -- 입력 전에 작동하도록 지정
    ON userTBL -- 트리거를 부착할 테이블
    FOR EACH ROW 
BEGIN
    IF NEW.birthYear < 1900 THEN
        SET NEW.birthYear = 0;
    ELSEIF NEW.birthYear > YEAR(CURDATE()) THEN
        SET NEW.birthYear = YEAR(CURDATE());
    END IF;
END // 
DELIMITER ;

INSERT INTO userTBL VALUES('AAA', '에이', 1877, '서울', '011', '1112222', 181, '2019-12-25');
INSERT INTO userTBL VALUES('BBB', '비이', 2977, '경기', '011', '1113333', 171, '2011-3-25');

SHOW TRIGGERS FROM sqlDB;

DROP TRIGGER userTBL_BeforeInsertTrg;
```



__ALTER TRIGGER 사용 불가__





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





## JOIN

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





# 데이터베이스 모델링 실습

>  게시판을 활용하여 회원들이 글을 쓰고, 댓글을 사용하고, 파일을 업로드할 수 있는 데이터베이스 모델링



객체: 사용자, 게시판, 작성글, 댓글, 파일

1. 사용자: 사용자id(pk), 패스워드, 이름, 가입일시, 성별, 연령, 작성글수, 댓글수 등 (이외 필요한 개인정보)

2. 게시판: 게시판명(pk), 게시글수 등

3. 작성글: 작성글번호(pk), 게시판명(fk), 글 제목, 작성자id(fk), 작성일시, 내용, 조회수, 댓글수, 댓글id

4. 댓글: 댓글id(pk), 작성글번호(fk), 작성자id(fk), 작성일시, 내용, 

5. 파일: 파일id(pk), 파일명, 작성글번호(fk), 업로드일시, 파일형식, 용량, 파일저장위치





1. __업무파악__

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



