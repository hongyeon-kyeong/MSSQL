# 정의 및 기능

오류메세지를 생성하고 세션에 대한 오류처리를 지정하는 구문

RAISEERROR는 sys.message 카탈로그 뷰에 저장된 사용자 정의 메세지를 참조하거나 동적으로 메세지를 작성할 수 있다.

메세지는 호출하는 애플리케이션 또는 연결된 TRY CATCH 구문의 CATCH 블록에 서버 오류 메세지로 반환된다.

# 구문

```sql
-- Syntax for SQL Server and Azure SQL Database  
  
RAISERROR ( { msg_id | msg_str | @local_variable }  
    { ,severity ,state }  
    [ ,argument [ ,...n ] ] )  
    [ WITH option [ ,...n ] ]
```

## 1. msg_id / msg_str / local_variable 중 하나 입력

### a. msg_id : sys.message 카탈로그 뷰에 저장된 사용자 정의 메세지를 참조

```sql
SELECT top 100 *
FROM SYS.messages
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ff3c3d3-299d-4ac2-a226-4b938fa551cf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ff3c3d3-299d-4ac2-a226-4b938fa551cf/Untitled.png)

- 사용자 정의 메세지 추가하는 법

    sys.message 카탈로그 뷰에 50005번 메세지 추가

    ```sql
    sp_addmessage @msgnum = 50005,  
                  @severity = 10,  
                  @msgtext = N'<\<%7.3s>>';  
    GO  
    RAISERROR (50005, -- Message id.  
               10, -- Severity,  
               1, -- State,  
               N'abcde'); -- First argument supplies the string.  
    -- The message text returned is: <<    abc>>.  
    GO  
    sp_dropmessage @msgnum = 50005;  
    GO
    ```

### b. msg_str or local_variable : 동적으로 메세지를 작성

- msg_str : C 표준 라이브러리의 printf 함수의 기능과 유사한 무자 대체를 지원

```sql
RAISERROR (N'This is message %s %d.', -- Message text.  
           10, -- Severity,  
           1, -- State,  
           N'number', -- First argument.  
           5); -- Second argument.  
-- The message text returned is: This is message number 5.  
GO
```

- local_variable

```sql
DECLARE @StringVariable NVARCHAR(50);  
SET @StringVariable = N'<\<%7.3s>>';  
  
RAISERROR (@StringVariable, -- Message text.  
           10, -- Severity,  
           1, -- State,  
           N'abcde'); -- First argument supplies the string.  
-- The message text returned is: <<    abc>>.  
GO
```

## 2. severity

메세지에 연결된 사용자 정의 심각도 수준. sp_addmessage를 사용해 만든 사용자 정의 메시지를 발생시키기 위해 msg_id 를 사용할 경우 RAISERROR에 지정된 심각도가 sp_addmessage에 지정된 심각도보다 우선한다.

20부터 25까지의 심각도는 치명적인 심각도입니다. 치명적인 심각도가 발생하면 메시지를 받은 다음 클라이언트 연결이 종료되고 오류 및 애플리케이션 로그에 오류가 기록됩니다.

```sql
RAISERROR (15600,-1,-1, 'mysp_CreateCustomer');

/* 결과 집합
Msg 15600, Level 15, State 1, Line 1   
An invalid parameter or option was specified for procedure 'mysp_CreateCustomer'.
*/
```

## 3. state (??)

0에서 255 사이의 정수입니다. 음수 값은 기본적으로 1입니다. 255보다 큰 값은 사용해서는 안 됩니다.

여러 위치에서 동일한 사용자 정의 오류가 발생하는 경우 각 위치의 고유 상태 번호를 사용하면 코드의 어떤 부분에서 오류가 발생하는지 찾는 데 도움이 됩니다.

[참고] TRY CATCH 사용하기

다음 코드 예제에서는 RAISERROR 블록 내의 TRY를 사용하여 관련 CATCH 블록으로 실행을 이동하는 방법을 보여 줍니다. 또한 RAISERROR 블록을 호출한 오류에 관한 정보를 반환하는 방법을 CATCH을 사용하여 보여 줍니다.

```sql
BEGIN TRY  
    -- RAISERROR with severity 11-19 will cause execution to   
    -- jump to the CATCH block.  
    RAISERROR ('Error raised in TRY block.', -- Message text.  
               16, -- Severity.  
               1 -- State.  
               );  
END TRY  
BEGIN CATCH  
    DECLARE @ErrorMessage NVARCHAR(4000);  
    DECLARE @ErrorSeverity INT;  
    DECLARE @ErrorState INT;  
  
    SELECT   
        @ErrorMessage = ERROR_MESSAGE(),  
        @ErrorSeverity = ERROR_SEVERITY(),  
        @ErrorState = ERROR_STATE();  
  
    -- Use RAISERROR inside the CATCH block to return error  
    -- information about the original error that caused  
    -- execution to jump to the CATCH block.  
    RAISERROR (@ErrorMessage, -- Message text.  
               @ErrorSeverity, -- Severity.  
               @ErrorState -- State.  
               );  
END CATCH;
```
