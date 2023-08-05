# Contents
SQL 기본 statement 정리, 날짜/시간/string 함수 정리, 팁, testdome 코드 연습 문제 및 풀이.  
  
자료는 대부분 사내 교육 자료를 참고하였고 SQL 문제는 testdome의 예제 문제입니다. 

---
# 🙊 1. Tips
### 📌 작성 순서랑 실행 순서 다르다
- 작성 순서: [SELECT - FROM - WHERE - GROUP BY - HAVING - ORDER BY]

    **SELECT** <DISTINCT> column-1<, …column-n> <AS> <column_name>  
    **FROM** table-1|view-1 <, ...table-n|view-n>  
    <**WHERE** expression>  
    <**GROUP BY** column-1 <, …column-n>>  
    <**HAVING** expression>  
    <**ORDER BY** column-1 <DESC><, …column-n>>  
    <**FETCH FIRST** n **ROWs ONLY**>  
  
- ⭐실행 순서⭐: [**FROM** - **WHERE** - **GROUP BY** - **HAVING** - **SELECT** - **ORDER BY**]

### 📌 NULL처리 함수: `coalesce`, `IFNULL`
- `**COALESCE**` 함수
    - `COALESCE(조건1, 조건2, ...)` = IF 조건1 IS NULL THEN 조건 2 ELSE ..

### 📌 날짜 포맷팅
🔥 SQLite는 `strftime()`, ' , MySQL은`date_format()`만 기억해도 됨.
- 포맷팅
    - MySQL: `date_format(날짜, 포맷)` 
    - SQLite: `strftime(포멧, 날짜1)`, `date(날짜)` (`date()`는 'YYYY-MM-DD'로 바꿔줌)
### 📌 날짜 차이
🔥 SQLite는 `julianday`, ' , MySQL은`datediff`만 기억해도 됨.  
- 날짜 차이 - 두 날짜 간의
    - `TIMESTAMPDIFF(DAY, 날짜1, 날짜2)+1`, `TIMESTAMPDIFF(MONTH, 날짜1, 날짜2)+1`  ⇒ MySQL
    - `julianday(DATE(날짜1)) - julianday(DATE(날짜2)) + 1` ⇒ SQLite
- 날짜 차이 - 특정 날짜 빼기
    - `ADDDATE(now(), INTERVAL +90 DAY)` 
    = `DATE_ADD(now(), INTERVAL +90 DAY)`  ⇒ MySQL
    - `strftime(포멧, 날짜1, '5 days')` , `date('now', '5 days')` 
    ⇒ SQLite

```sql
strftime('%Y', '2023-03-03') => '2023'
strftime('%Y', '2023-03-03', '1 years') => 2024
julianday(date('2023-03-02')) - julianday(date('2023-03-05')) = -2 
```

### 📌 `NULL` ⭐
- `max`, `min`, `avg`, `sum`은 `NULL`을 비교 대상으로 삼지 않는다.   
 => 상관없이 결과가 나옴
- 함수 안쓰고  +, -, *, /면 `NULL`
- `count`
    - `count(*)` 테이블의 **모든** 행을 계산하지만
    - `count(column)`해당 열의 **NULL이 아닌 값**만 계산함  
    => select count(column)하면 null은 안세는데  
       select distinct column하면 null도 나온다
    - BUT 공백(’’) count되니 안되게 하려면 Null로 바꾸거나 **WHERE** 조건 걸기
- 공백(‘’)랑 다름
- 둘 다 잡으려면 WHERE (name <> ‘’) AND (name IS NOT NULL)
- - 더하거나 곱하거나 나누면 NULL

- 0으로 나누기 → NULL
- 0을    나누기 → 0
- string 0으로 나누기 / 0을 string으로 나누기 → NULL
- 공백(’’) 0으로 나누기 / 0을 공백으로 나누기 → NULL

### 기타
- SELECT SUBSTRING_INDEX(기준컬럼, delimiter, 몇개) ⇒ MySQL
- substr + instr 쓰기

### 📌 **WHERE** vs **HAVING**
**WHERE**랑 **HAVING** 둘 다 조건 statement인데, 

1. 어디에 적용되는 조건인지에 따라
    - **WHERE** 절은 그룹핑 되기 전에 적용되는 조건
        
        ```sql
        SELECT workdept
             , avg(salary)
          FROM workdept
         WHERE salary > 20000 -- salary > 2000되는 애들만 남아서 workdept로 그룹핑
        -- avg(salary) 안됨
        ```
        
    - **HAVING** 절은 그룹핑 된 후 그룹에 적용되는 조건
        
        ```sql
        SELECT workdept
             , avg(salary)
          FROM workdept
        HAVING mean(salary) > 20000 -- workdept로 그룹핑된 그룹 중에
        														-- 그룹 평균이 20000 넘는 그룹만 조회
        ```
        
2. 집계함수 사용 가능 여부에 따라
    - **WHERE**은 **GROUP BY** 전이라 집계함수로 조건 걸 수 없고
    - **HAVING**은 **GROUP BY** 후라 집계함수 사용해 조건 가능 (**GROUP BY로 그룹핑된** 그룹에 대해)

- **WHERE**은 테이블 전체 레코드 검색됨 (**FROM**만 영향)
- **GROUP BY**는 **FROM** + **WHERE**의 실행 결과를 그룹으로 분류
- SQL은 테이블명, 컬럼명 대소문자 구분하지 않음
- NULL도 하나의 Instance..? (애매)
    - DISTINCT하면 NULL도 하나의 종류로 print 되니까 NULL 없이 하려면 where column-1 NOT NULL 추가
        
        [https://www.notion.so/SQL-5172e0ca16d44e16b1a387546c48cf01?pvs=4#36e49f41ed1f4bf5b0fc905e46d0d2b2](https://www.notion.so/SQL-5172e0ca16d44e16b1a387546c48cf01?pvs=21)
        
    - 근데 또 count()하면 안잡힘
- 그룹 함수 사용시 NULL을 반환하더라도 NULL이 한건의 레코드가 되므로 결과값에 상관없이 EXISTS → True, NOT EXISTS → False가 됨 (교재 p.131)
    
    ```sql
    SELECT * 
      FROM edu.employee
     WHERE NOT EXISTS (SELECT sum(empno)
                         FROM eud.employee
                        where sex='T')
    ```

---
# 2. SQL 기본
## `SELECT` statement
- `문자열상수`  
    작은 따옴표 안에 문자열 지정 
    ```sql
    SELECT 'I"m'  --여기
        , '급여는'
    FROM edu.employee
    ;
    ```

- `Alias`  
    `AS`는 생략 가능; 따옴표 안써도 되는데 공백이나 특수문자를 포함하거나 대소문자를 구분하는 경우 큰따옴표(” “)를 반드시 써줘야함.
    단, WHERE 절에서는 Alias 사용할 수 없음

    ```sql
    SELECT firstname AS 이름
        , lastname  AS “성1”  --여기
    FROM edu.employee
    ;
    ```

- `LIMIT`  
  상위 n개의 로우만 **프린트** 할 때  
  = 다 끝나고 진짜 프린트할때만 영향        
  = **ORDER BY** 정렬 후 결과값 상위 n개의 열만 표시   

    인덱스가 0부터 -> LIMIT 0, 5해야 첫번째 레코드부터 나옴

    ```sql
    --group by 한 뒤 첫번째 관측치만 나오게 하려면
    --1. LIMIT 쓰기
    SELECT DATETIME
    FROM AA 
    ORDER BY DATETIME DESC
    LIMIT 0  -- OR FETCH FIRST 1 ROWS ONLY
    ;

    --2. SUBQUERY
    SELECT DATETIME
    FROM (SELECT * FROM AA ORDER BY DATETIME DESC)
    WHERE ROWNUM=1

    --3. 오답
    --[FROM - WHERE - GROUP BY - HAVING - SELECT - ORDER BY]순서라서
    --테이블AA에서 첫번째 행 가져온 뒤 DATETIME SELECT하고 정렬함
    SELECT DATETIME
    FROM AA
    WHERE ROWNUM=1
    ORDER BY DATETIME DESC
    --ref: https://dobby-codeup.tistory.com/7

    ```

    SQLite, MySQL; LIMIT (rownum, FETCH FIRST 없음)  
    DB2 - FETCH FIRST 10 ROWS ONLY

- `||`  
    문자형 데이터 연결할 때는 연결 연산자인 || (파이프 두개) 이용 → DB2에만 먹는 듯  
    `CONCAT(문자열1, 문자열2, ...)` 쓰기

- `CASE WHEN`
    ```sql
    SELECT firstname
        , workdept
        , CASE WHEN substr(workdept, 1, 1) = 'A' THEN 100
               WHEN substr(workdept, 1, 1) = 'B' THEN NULL
               ELSE trunc(salary*0.1, 1) -- 여기집중. 여기에 함수 쓸 수 있음
           END grade
    FROM edu.employee
    ;
    ```

---

## `ORDER BY` statement
- 대소 비교 기준: 공백문자 < 기호 < 숫자 < 대문자 < 소문자 <  한글 < NULL
- 컬럼번호로 정렬 가능 `SELECT * FROM AA ORDER BY 2;`
- 수식 컬럼 정렬 가능 `SELECT * FROM AA ORDER BY salary + bonus;`
- SELECT절의 Alias로 정렬 가능
`SELECT dept, avg(salary) AS average FROM AA ORDER BY average`
⇒ avg 함수로 집계하고 그 기준으로 sorting

---

## `WHERE` statement

- 주요 연산자
    | 종류 | 조건 연산자 |
    | --- | --- |
    | 비교 | =, <>, >, <, <=, >= |
    | 논리 | AND OR NOT |
    | IN | IN NOT IN |
    | BETWEEN | BETWEEN NOT BETWEEN |
    | LIKE | LIKE NOT LIKE |
    | IS NULL | IS NULL IS NOT NULL |
- 우선순위
    
    OR < AND < NOT
    
    ⇒ ~~외우기 귀찮으니~~ 괄호를 이용해 연산의 우선 순위 지정
    

- `IN` `NOT IN` `BETWEEN` `NOT BETWEEN` `LIKE` `IS NULL` `IS NOT NULL` 예시

    ```sql
    SELECT *
    FROM AA
    WHERE SUBSTR(C1, 1, 1) BETWEEN 1 AND 5
    -- WHERE SUBSTR(C1, 1, 1) NOT BETWEEN 'A' AND 'C'
    -- WHERE SUBSTR(C1, 1, 1) BETWEEN 1 AND 5

    -- WHERE C1 LIKE 'A%' -- 'A', 'ADFJIEFKD', 'A가나'
    -- WHERE C1 LIKE 'A_' -- 'AB' (A 안됨)
    -- WHERE C1 LIKE 'A_B%' -- 'AIBIIIIISI209'
    -- WHERE C1 LIKE '2023%' -- '2023-01-01'

    -- WHERE C1 IS NOT NULL
    -- WHERE C1 IS NULL
    ```

---

## `GROUP BY` statement

-  **SELECT**에서 조회하는 컬럼에 대해서만 **GROUP BY** 기준으로 삼을 수 있음  
    ⇒ **SELECT**에 있는거 다 쓰기
    
- 어떤 인스턴스에 대해 집계되는걸까?   
    ⇒ **FROM** table **Where** 조건으로 조회된 인스턴스에 대해  
    cf) **실행 순서**  
    [**FROM** - **WHERE** - **GROUP BY** - **HAVING** - **SELECT** - **ORDER BY**]
    
- 함수  
    ⇒ `SUM` `AVG` `MIN` `MAX` `COUNT`
    
- 어떻게 써?  
    ⇒ **GROUP BY**에 들어간 거 빼고 SELECT에 있는거 다 써. ‼️CASE WHEN도 다 넣어
    

---

## `HAVING` statement
- **GROUP BY**로 만들어진 그룹에 대한 조건 (vs **WHERE**: **GROUP BY** 되기 전)
- **HAVING**절에 사용된 그룹 함수가 반드시 **SELECT** 절에서 지정될 필요 없음
    ```sql
    SELECT workdept
         , AVG(salary)
      FROM workdept
     GROUP BY workdept
    HAVING AVG(bonus) > 60000
    ```
    
- 기본: 조건식에 그룹 함수 지정
    ```sql
    SELECT workdept
         , AVG(salary)
      FROM workdept
     GROUP BY workdept
    HAVING AVG(salary) > 60000
    ```
---
## `JOIN`
- **WHERE**절이 **JOIN**보다 먼저 적용
- 내부조인은 `INNER JOIN ~ ON` 을 사용하지 않고 표현 가능 (주로 이 형식 많이 사용함, 3개도 가능)
    ```sql
    SELECT *
      FROM employee a, department b
     WHERE a.workdept = b.workdept
    ```
---
## `Subquery`

```sql
SELECT * 
  FROM employee
 WHERE salary = (SELECT min(salary) FROM employee WHERE workdept='E01')

SELECT dept
     , count(1)
  FROM employee
HAVING AVG(salary) > (SELECT min(salary)FROM )
```

---
---
# 3. ⭐날짜/시간/string 관련 함수⭐
## 1) 포멧 (공통)
**날짜 관련** 
| 형식 | print |
| --- | --- |
| %Y | yyyy (연도, 4자리) |
| %y | yy (연도, 뒤에 2자리) |
| %M | August (월) |
| %b | Aug (월, Jan ~ Dec) |
| %m | 08 (월, 01 ~ 12) |
| %d | dd (일, 00~31) |

**시간 관련**
| 형식 | print |
| --- | --- |
| %H | 시간 24시간(00~23) |
| %h | 시간 12시간(00~12) |
| %i | 분 (00~59) |
| %s | 초 (00~59) |

ref  
https://www.w3schools.com/sql/func_mysql_date_format.asp  
https://devjhs.tistory.com/89

---
## 2) `SQLite`
### 1. 함수  

| 함수 | 내용 |
| --- | --- |
| strftime(포맷, 시간표현, 옵션) | 지정한 포맷으로 값 반환 |
| date(시간표현, 옵션, …) | ‘YYYY-MM-DD’ 형식 반환 |
| time(시간표현, 옵션, …) | 'HH:MM:SS’ 형식반환 |
| datetime(시간표현, 옵션, …) | ‘YYYY-MM-DD HH:MM:SS’ 형식 반환 |

### 2. 예시 코드
```sql
 select strftime('%Y-%m-%d', CURRENT_TIMESTAMP)  --상위3개같음
      , date(CURRENT_TIMESTAMP)
      , CURRENT_DATE  --YYYY-MM-DD
      , date('now')

      , strftime('%Y-%m-%d', 'now', '5 days')
      , strftime('%Y-%m-%d', CURRENT_DATE, 'start of year')
  from demo
 limit 1
```

### 3. 시간표현
  - column
  - `CURRENT_DATE`
  - `CURRENT_TIME`
  - `CURRENT_DATETIME`

### 4. 옵션 - 날짜, 시간에 더하고 빼기
  - 위에 함수에 옵션 추가해서 가능
  - `'n days'`   따음표 꼭 해줘야함
  - 예시: `date('now', '5 days')` `date('now', '-3 years')`
      - now에서 5일 더하거나 3년 뺀 ‘YYYY-MM-DD’ 형식 반환
  - 옵션 종류
      - `days`, `months`, `years`, `hours`, `minutes`, …
      - `'start of year'`

    ```sql
    select strftime('%Y-%m-%d', 'now', '5 days')
        , strftime('%Y-%m-%d', CURRENT_DATE, 'start of year')
    from demo
    limit 1
    ```

### 5. 시간차이  
`JULIANDAY(시간표현)` 으로 계산해야함

```sql
--일수 계산 -> DATE 함수도 써야함.
--DATE 안씌우면 소수점도 나올 수 있는데 이건 하루가 충분히 안된다면. 시간도 있는경우
SELECT JULIANDAY() - JULIANDAY()
...
```

### 6. 특정 부분만 추출
`strftime('%m', '2023-08-01')` = ‘08’  
`strftime('%Y', '2023-08-01')` = ‘2023’

### 7. string 함수 (문자열 조작 함수)

| lower(x) | lower(’ABC’) = abc |
| --- | --- |
| upper(x) | upper(’AbC’) = ABC |
| length(x) | length(’abc’) = 3 |
| trim(x, y) | 문자열 x의 양 끝에서 y를 제거한 결과
trim(’abc’, ‘c’) = ‘ab’
trim(’abc’, ‘d’) = ‘abc’
trim(’abc  ‘, ‘ ‘) = ‘abc’ |
| ltrim(x, y) | 문자열 x의 왼쪽 끝에서 y를 제거한 결과 |
| rtrim(x, y) | 문자열 x의 오른쪽 끝에서 y를 제거한 결과 |
| substr(x, y, [z]) | 문자열 x의 y번째 인덱스에서 z개 추출 |
| replace(x, y, z) | 문자열 x 중에서 y와 일치하는 문자열을 z로 교체 |
| instr(x, y) | 문자열 x 중에서 y랑 처음 일치하는 곳의 인덱스 리턴 |
|  | instr(’www@naver.com’, ‘@’) = 4 |
|  | substr(’www@naver.com’, 1, instr(’www@naver.com’, ‘@’)) = ‘www’ |

### 8. 띄어쓰기 기준으로 쪼개기
MySQL의 substring_index처럼 delimiter 기준으로 쪼개주는 함수는 없다.
```sql
substr(address, 1, instr(address, ' '))
--instr: 특정 문자열이 나오는 첫번째 Index return
instr(address, ' ') -- IF address='A real human' then 2
--substr: index로 string 쪼개기
substr(문자열, 시작 위치, 문자의 길이)
--or
substr(문자열, 시작위치)
```

### Reference

- https://thinking-jmini.tistory.com/7

---
## 3) `MySQL`  
### 1. 포맷팅
- `DATE_FORMAT(날짜, 형식)`: 날짜를 지정한 형식으로 출력  
    ```sql
    DATE_FORMAT(날짜, 형식): 날짜를 지정한 형식으로 출력
    DATE_FORMAT(now(), '%Y-%m-%d')  --'yyyy-mm-dd'
    ```

### 2. 시간값 더하고 빼기
| 함수 | 내용 |
| --- | --- |
| ADDDATE(now(), INTERVAL +90 DAY)
= DATE_ADD(now(), INTERVAL +90 DAY) | 날짜 더하기 (빼기는 -90) |
| DATEDIFF(날짜1, 날짜2) | 일 차이 |
| TIMESTAMPDIFF(단위, 날짜1, 날짜2) | 단위: year, month, week, day etc |
| DATE_FORMAT(now(), '%Y-%m-%d') | 날짜 출력 형식 설정 |
| DATE YEAR MONTH DAY | 특정 부분만 추출 |

```sql
SELECT DATE_ADD(now(), INTERVAL -90 DAY)
        , DATE_ADD(now(), INTERVAL +90 DAY)
        , DATE_ADD(now(), INTERVAL +2 MONTH)
        , TIMESTAMPDIFF(YEAR, '2023-03-02', '2024-05-23') = 1
        , TIMESTAMPDIFF(YEAR, '2023-03-02', '2023-05-02') = 0
        , MONTH('2023-09-10') --9
        , DATE_FORMAT('2022-08-01 00:00:00', '%Y-%m-%d') --‘2022-08-01’
```

참고) `Oracle`, `DB2`: `TO_CHAR(sysdate, format)`

### 3. 띄어쓰기 기준으로 split

```sql
SELECT SUBSTRING_INDEX(기준컬럼, delimiter, 몇개)
--예시
SELECT SUBSTRING_INDEX(address, ' ', 2)  -- 경기도 수원시
     , SUBSTRING_INDEX(address, ' ', 1)  -- 
```
---
## 기타
### 1. 올림, 반올림, 사칙연산
- `round(column,` ‼️`**표시될 자리수**`‼️`)`:`round(39.125, 2)=39.16` 둘째자리까지 표시(셋째자리에서 반올림)

- `trunc`: 버림. ex) `trunc(398.233, 2)=300`
- `mod(column, n)`: n으로 나눈 나머지. ex) `mod(10, 2)=2`
- `/`: 몫
- `%`: 나머지

### 2. 데이터 유형
1. `VARCHAR(size)`: Varing character
2. `CHAR`
3. `NUMBER`
4. `INTEGER`
5. `DATE`
6. `LONG`
...

---
## 4. DDL (Data Definition Language)
💡 FOREIGN KEY 지정 시 references (reference ❌)

### 1. `CREATE`

```sql
CREATE TABLE 테이블이름 (
	열이름  데이터타입 [DEFAULT 값] [NOT NULL] [PRIMARY KEY],
    열이름  데이터타입 [DEFAULT 값] [NOT NULL] [FOREIGN KEY REFERENCES 테이블(컬럼명),
    [또는 PRIMARY KEY (열 리스트-1, [열 리스트-2]),
    [또는 FOREIGN KEY (열 리스트) REFERENCES 테이블이름 (열 리스트) [ON DELETE CASCADE]]
)
;

--예시
CREATE TABLE employees (
	id INTEGER NOT NULL PRIMARY KEY,
	managerId INTEGER,
    name VARCHAR(30) NOT NULL,
	FOREIGN KEY (managerId) REFERENCES employees(id)
)
;

INSERT INTO employees(id, managerId, name) VALUES (1, NULL, 'John');
INSERT INTO employees(id, managerId, name) VALUES (2, 1, 'Mike');
```

- `ON DELETE CASCADE`  
FOREIGN KEY 만들때 쓸 수 있는 명령어로 참조하는 키의 row가 지워지면 같이 지워지게 하는 명령어. 즉 foreign key로 연결된 row를 한번에 지우는 방법이다.

- 다른 테이블 정보를 활용한 테이블 생성(테이블 복사는 DBMS마다 다를 수 있음)

```sql
CREATE TABLE 테이블이름 AS SELECT 문;
```

### 2. `UPDATE`
```sql
UPDATE [테이블]
   SET [열] = '변경할값'
 WHERE [조건]

--예시
UPDATE enrollments SET year = 2015 WHERE id >= 20
UPDATE enrollments SET year = 2015 WHERE year IS NULL
```

### 3. `DELETE` `DROP`
```sql
DELETE enrollments WHERE year = 2015;
DROP TABLE enrollments;
```

---
---

## Testdome 예제 풀기

- 솔루션: https://gist.github.com/mindyng/028a84855cb05f27d6a8feb1b42df744

- 예제1  `#group by`

    ```sql
    --TABLE sessions
    --	id INTEGER PRIMARY KEY,
    --  userId INTEGER NOT NULL,
    --  duration DECIMAL NOT NULL
    --Write a query that select userId and average session duration for each user
    --whoe has more than one sessions.

    --답1
    WITH TEMP AS (
    SELECT USERID
        , COUNT(1) AS CNT
        , AVG(DURATION) AS AVERAGEDURATION
    FROM SESSIONS
    GROUP BY 1
    )

    SELECT USERID
        , AVERAGEDURATION
    FROM TEMP
    WHERE CNT > 1

    -----------------------------------------
    --답2
    WITH TEMP AS (
    SELECT USERID
        , COUNT(1) AS CNT
        , AVG(DURATION) AS AVERAGEDURATION
    FROM SESSIONS
    GROUP BY 1
    )

    SELECT USERID
        , AVERAGEDURATION
    FROM TEMP
    WHERE CNT > 1
    ```

- 예제 2 `#NOT NULL in distinct`

    ```sql
    -- Suggested testing environments
    -- For MS SQL:
    -- https://sqliteonline.com/ with language set as MS SQL
    -- For MySQL:
    -- https://www.db-fiddle.com/ with MySQL version set to 8
    -- For SQLite:
    -- http://sqlite.online/
    -- Put the following without '--' at the top to enable foreign key support in SQLite.
    -- PRAGMA foreign_keys = ON;

    -- Example case create statement:
    CREATE TABLE employees (
    id INTEGER NOT NULL PRIMARY KEY,
    managerId INTEGER, 
    name VARCHAR(30) NOT NULL,
    FOREIGN KEY (managerId) REFERENCES employees(id)
    );

    INSERT INTO employees(id, managerId, name) VALUES(1, NULL, 'John');
    INSERT INTO employees(id, managerId, name) VALUES(2, 1, 'Mike');

    -- Expected output (in any order):
    -- name
    -- ----
    -- Mike

    -- Explanation:
    -- In this example.
    -- John is Mike's manager. Mike does not manage anyone.
    -- Mike is the only employee who does not manage anyone.

    --Answer
    SELECT NAME
    FROM EMPLOYEES
    WHERE ID NOT IN (SELECT DISTINCT MANAGERID FROM EMPLOYEES WHERE MANAGERID NOT NULL)
    ```

- 예제3 `#Create table` 

    ```sql
    --users, roles 테이블 있을때 아래의 제약사항 만족하는 usersRoles 테이블 정의
    --Modify the provided SQL create table statement so that:
    --The usersRoles table should contain the mapping between
    --each user and their roles.
    --Each user can have many roles, and each role can have many users.
    -- 1 Only users from the users table can exist within usersRoles.
    -- 2 Only roles from the roles table can exist within usersRoles.
    -- 3 A user can only have a specific role once.

    CREATE TABLE users (
    id INTEGER NOT NULL PRIMARY KEY,
    userName VARCHAR(50) NOT NULL
    );

    CREATE TABLE roles (
    id INTEGER NOT NULL PRIMARY KEY,
    role VARCHAR(20) NOT NULL
    );

    INSERT INTO users(id, userName) VALUES(1, 'Steven Smith');
    INSERT INTO users(id, userName) VALUES(2, 'Brian Burns');

    INSERT INTO roles(id, role) VALUES(1, 'Project Manager');
    INSERT INTO roles(id, role) VALUES(2, 'Solution Architect');

    -- Improve the create table statement below:
    CREATE TABLE usersRoles (
    userId INTEGER,
    roleId INTEGER
    );

    -- The statements below should pass.
    INSERT INTO usersRoles(userId, roleId) VALUES(1, 1);
    INSERT INTO usersRoles(userId, roleId) VALUES(1, 2);
    INSERT INTO usersRoles(userId, roleId) VALUES(2, 2);

    -- The statement below should fail.
    INSERT INTO usersRoles(userId, roleId) VALUES(2, NULL);

    --Answer
    CREATE TABLE usersRoles(
        userId INTEGER NOT NULL,
        roleId INTEGER NOT NULL,
        PRIMARY KEY (userId, roleId),
        FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE,
        FOREIGN KEY (roleId) REFERENCES roles(id) ON DELETE CASCADE
    )

    --오답; 왜 오답인지 모르겠다 -> 찾음!! 고침.
    --error case: Some regions have no employees, or all employees have no sales: Exception.
    WITH temp1 as (
    SELECT n10.id as employeeId
        , n10.stateId 
        , n30.regionId as regionId
        , n20.amount
    FROM employees as n10
    LEFT JOIN sales as n20 on (n10.id = n20.employeeId)
    LEFT JOIN states as n30 on (n10.stateId = n30.id)
    LEFT JOIN regions as n40 on (n30.regionId = n40.id)
    )
    , temp2 as (
    SELECT regionId
        , SUM(amount)/COUNT(DISTINCT employeeId) as average
    --count(column)해서 0으로 나누는 경우없기도 하고 이미 amount 있는 애들 대상으로 하니 
    --sum도 count도 0이 아니라 IFNULL 안해도 됨.
    --하지만 밑에 region이랑 붙일때는 없는 애들도 있을텐니 안붙는 경우에 대해 IFNULL 해야함.
        FROM temp1
    GROUP BY 1
    )
    SELECT n10.name
        , IFNULL(n20.average, 0) as average
        , (SELECT MAX(average) FROM temp2) - IFNULL(n20.average, 0) as difference
    --오답; join할 때 안붙은거는 ''이 아니라 null
    --     , CASE WHEN n20.average = '' THEN 0 ELSE n20.average 
    --        END as average
    --     , CASE WHEN n20.average = '' THEN 0 ELSE (SELECT MAX(average) FROM temp2) - n20.average
    --        END as difference
    FROM regions n10 
        LEFT JOIN temp2 n20 on (n10.id = n20.regionId)
    --다른 답
    WITH TEMP AS (
    SELECT n10.name as regionName
        , IFNULL(IFNULL(sum(amount), 0) / count(distinct n30.id),0) as average
    FROM regions n10
        LEFT JOIN states n20 ON (n10.id = n20.regionId)
        LEFT JOIN employees n30 ON (n20.id = n30.stateId)
        LEFT JOIN sales n40 ON (n30.id = n40.employeeId)
    GROUP BY 1
    )
    select regionName
        , average
        , (select max(average) from temp) - average as difference
    from temp
    ```

- 예제4 `# `` and NULL` `# They are totally **different**!!`  
    **문제**
    ```sql
    -- Suggested testing environments
    -- For MS SQL:
    -- https://sqliteonline.com/ with language set as MS SQL
    -- For MySQL:
    -- https://www.db-fiddle.com/ with MySQL version set to 8
    -- For SQLite:
    -- http://sqlite.online/
    -- Put the following without '--' at the top to enable foreign key support in SQLite.
    -- PRAGMA foreign_keys = ON;
    
    -- Example case create statement:
    CREATE TABLE regions(
      id INTEGER PRIMARY KEY,
      name VARCHAR(50) NOT NULL
    );
    
    CREATE TABLE states(
      id INTEGER PRIMARY KEY,
      name VARCHAR(50) NOT NULL,
      regionId INTEGER NOT NULL,
      FOREIGN KEY (regionId) REFERENCES regions(id)
    );
    
    CREATE TABLE employees (
      id INTEGER PRIMARY KEY,
      name VARCHAR(50) NOT NULL,
      stateId INTEGER NOT NULL,
      FOREIGN KEY (stateId) REFERENCES states(id)
    );
    
    CREATE TABLE sales (
      id INTEGER PRIMARY KEY,
      amount INTEGER NOT NULL,
      employeeId INTEGER NOT NULL,
      FOREIGN KEY (employeeId) REFERENCES employees(id)
    );
    
    INSERT INTO regions(id, name) VALUES(1, 'North');
    INSERT INTO regions(id, name) VALUES(2, 'South');
    INSERT INTO regions(id, name) VALUES(3, 'East');
    INSERT INTO regions(id, name) VALUES(4, 'West');
    INSERT INTO regions(id, name) VALUES(5, 'Midwest');
    
    INSERT INTO states(id, name, regionId) VALUES(1, 'Minnesota', 1);
    INSERT INTO states(id, name, regionId) VALUES(2, 'Texas', 2);
    INSERT INTO states(id, name, regionId) VALUES(3, 'California', 3);
    INSERT INTO states(id, name, regionId) VALUES(4, 'Columbia', 4);
    INSERT INTO states(id, name, regionId) VALUES(5, 'Indiana', 5);
    
    INSERT INTO employees(id, name, stateId) VALUES(1, 'Jaden', 1);
    INSERT INTO employees(id, name, stateId) VALUES(2, 'Abby', 1);
    INSERT INTO employees(id, name, stateId) VALUES(3, 'Amaya', 2);
    INSERT INTO employees(id, name, stateId) VALUES(4, 'Robert', 3);
    INSERT INTO employees(id, name, stateId) VALUES(5, 'Tom', 4);
    INSERT INTO employees(id, name, stateId) VALUES(6, 'William', 5);
    
    INSERT INTO sales(id, amount, employeeId) VALUES(1, 2000, 1);
    INSERT INTO sales(id, amount, employeeId) VALUES(2, 3000, 2);
    INSERT INTO sales(id, amount, employeeId) VALUES(3, 4000, 3);
    INSERT INTO sales(id, amount, employeeId) VALUES(4, 1200, 4);
    INSERT INTO sales(id, amount, employeeId) VALUES(5, 2400, 5);
    
    -- e.g. 'Minnesota' is the only state under the 'North' region.
    -- Total sales made by employees 'Jaden' and 'Abby' for the state of 'Minnesota' is 5000 (2000 + 3000)
    -- Total employees in the state of 'Minnesota' is 2
    -- Average sales per employee for the 'North' region = Total sales made for the region (5000) / Total number of employees (2) = 2500
    -- Difference between the average sales of the region with the highest average sales ('South'),
    -- and the average sales per employee for the region ('North') = 4000 - 2500 = 1500.
    -- Similarly, no sale has been made for the only state 'Indiana' under the region 'Midwest'.
    -- So the average sales per employee for the region is 0.
    -- And, the difference between the average sales of the region with the highest average sales ('South'),
    -- and the average sales per employee for the region ('Midwest') = 4000 - 0 = 4000.
    
    -- Expected output (rows in any order):
    -- name     average   difference
    -- -----------------------------
    -- North	2500	  1500
    -- South 	4000	  0
    -- East		1200   	  2800
    -- West		2400	  1600
    -- Midwest  	0         4000
    ```
    
    **풀이**
    ```sql
    WITH temp1 as (
    SELECT n10.id as employeeId
        , n10.stateId 
        , n30.regionId as regionId
        , n20.amount
    FROM employees as n10
    LEFT JOIN sales as n20 on (n10.id = n20.employeeId)
    LEFT JOIN states as n30 on (n10.stateId = n30.id)
    LEFT JOIN regions as n40 on (n30.regionId = n40.id)
    )
    , temp2 as (
    SELECT n10.id as regionId
        , n10.name
        , IFNULL(SUM(n20.amount) / COUNT(DISTINCT n20.employeeId), 0) as average
        FROM regions n10
            LEFT JOIN temp1 n20 on (n10.id = n20.regionId)
    GROUP BY 1
    )
    SELECT name
        , CASE WHEN average = '' THEN 0 ELSE average END as average
        , CASE WHEN average = '' THEN 0 ELSE (SELECT MAX(average) FROM temp2) - average
        END as difference
    FROM temp2
    ```

- 예제5 `INNER JOIN? LEFT JOIN?`

    ```sql
    -- Suggested testing environments
    -- For MS SQL:
    -- https://sqliteonline.com/ with language set as MS SQL
    -- For MySQL:
    -- https://www.db-fiddle.com/ with MySQL version set to 8
    -- For SQLite:
    -- http://sqlite.online/
    -- Put the following without '--' at the top to enable foreign key support in SQLite.
    -- PRAGMA foreign_keys = ON;

    -- Example case create statement:
    CREATE TABLE sellers (
    id INTEGER NOT NULL PRIMARY KEY,
    name VARCHAR(30) NOT NULL,
    rating INTEGER NOT NULL
    );

    CREATE TABLE items (
    id INTEGER NOT NULL PRIMARY KEY,
    name VARCHAR(30) NOT NULL,
    sellerId INTEGER,
    FOREIGN KEY (sellerId) REFERENCES sellers(id)
    );

    INSERT INTO sellers(id, name, rating) VALUES(1, 'Roger', 3);
    INSERT INTO sellers(id, name, rating) VALUES(2, 'Penny', 5);

    INSERT INTO items(id, name, sellerId) VALUES(1, 'Notebook', 2);
    INSERT INTO items(id, name, sellerId) VALUES(2, 'Stapler', 1);
    INSERT INTO items(id, name, sellerId) VALUES(3, 'Pencil', 2);

    -- Expected output (in any order):
    -- Item      Seller
    -- ----------------
    -- Notebook  Penny
    -- Pencil    Penny

    --Answer
    SELECT n20.name as itemName
        , n10.name as sellerName
    FROM sellers n10 
        INNER JOIN items n20 ON (n10.id = n20.sellerId)
    --     LEFT JOIN은 오답!!!!!! (...)
    WHERE n10.rating > 4
    ORDER BY n10.rating DESC
    ```

- 예제6 `UNION and DISTINCT`

    ```sql
    --Problem
    -- Suggested testing environments
    -- For MS SQL:
    -- https://sqliteonline.com/ with language set as MS SQL
    -- For MySQL:
    -- https://www.db-fiddle.com/ with MySQL version set to 8
    -- For SQLite:
    -- http://sqlite.online/

    -- Example case create statement:
    CREATE TABLE dogs (
    id INTEGER NOT NULL PRIMARY KEY, 
    name VARCHAR(50) NOT NULL
    );

    CREATE TABLE cats (
    id INTEGER NOT NULL PRIMARY KEY, 
    name VARCHAR(50) NOT NULL
    );

    INSERT INTO dogs(id, name) values(1, 'Lola');
    INSERT INTO dogs(id, name) values(2, 'Bella');
    INSERT INTO cats(id, name) values(1, 'Lola');
    INSERT INTO cats(id, name) values(2, 'Kitty');

    -- Expected output (in any order):
    -- name     
    -- -----
    -- Bella    
    -- Kitty    
    -- Lola

    --Answer
    SELECT DISTINCT name --distinct 꼭 해줘야함
                        --UNION 하더라도 dogs and cats에 동일 이름이 있을수도 있으니
    FROM 
    (
    SELECT * FROM dogs
    UNION
    SELECT * FROM cats
    )
    WHERE (name NOT NULL) or (name <> '')
    ```

- 예제7
    **Brenda가 0이 아니라 1인 이유**  
    - `COUNT(*)`는 모든 행을 count하기 때문에 left join으로 하나도 안붙어도 카운팅된다.  
    - `COUNT(column명)` 이었으면 null은 카운팅하지 않아서 0

---
---

## 실습: 프로그래머스 + Hackerrank
