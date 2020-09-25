---
author: 忠宇
---

# CTE

## 介紹
-   通用資料表運算式: 定義在單一 SELECT、INSERT、UPDATE、DELETE、MERGE、CREATE VIEW 陳述式的執行範圍內
-   遞迴通用資料表運算式: 當通用資料表運算式指向本身的參考

## 語法
-   syntaxsql

    ```sql
        [ WITH <common_table_expression> [ ,...n ] ]  
          
        <common_table_expression>::=  
            expression_name [ ( column_name [ ,...n ] ) ]  
            AS  
            ( CTE_query_definition )
    ```

## 引數
-   expression_name: expression_name 與相同 WITH <common_table_expression> 子句中所定義任何其他通用資料表運算式的名稱不得相同，但 expression_name 可與基底資料表或檢視同名。
-   column_name: 在一般資料表運算式中，指定資料行名稱。 在單一 CTE 定義內，名稱不能重複。 指定的資料行名稱數目必須與 CTE_query_definition 的結果集資料行數目相符。 只有在查詢定義提供了所有結果資料行的個別名稱時，資料行名稱清單才是選擇性的。
-   CTE_query_definition: 指定其結果集擴展一般資料表運算式的 SELECT 陳述式。 除了 CTE 不能定義另一個 CTE 之外，CTE_query_definition 的 SELECT 陳述式必須符合建立檢視表的相同需求。
-   如果定義了多個 CTE_query_definition，就必須由下列設定運算子來聯結查詢定義：UNION ALL、UNION、EXCEPT 或 INTERSECT
    
## 備註(細節太多，請自行查閱官方文件)
-   通用資料表運算式
    -   CTE_query_definition 中不允許使用下列項目：
        -   ORDER BY (除非指定了 TOP 子句)
        -   INTO
        -   含有查詢提示的 OPTION 子句
        -   FOR BROWSE

-   遞迴通用資料表運算式
    -   CTE_query_definition 中不允許使用下列項目:
        -   SELECT DISTINCT
        -   GROUP BY
        -   PIVOT
        -   HAVING
        -   純量彙總
        -   TOP
        -   LEFT、RIGHT、OUTER JOIN (允許使用 INNER JOIN)
        -   子查詢
        -   適用於對 CTE_query_definition 內 CTE 之遞迴參考的提示

## 範例
-   簡單的通用資料表運算式
    ```sql
        -- Define the CTE expression name and column list.  
        WITH P (Id, Pid, BirthYear)  
        AS  
        -- Define the CTE query.  
        (  
            SELECT Id, Pid, YEAR(Birthday) AS BirthYear  
            FROM [SbRe].[AssociatedPerson]
            WHERE Pid IS NOT NULL AND Birthday IS NOT NULL
        )  
        -- Define the outer query referencing the CTE name.  
        SELECT P.BirthYear, P.Pid FROM P
        GROUP BY P.BirthYear, P.Pid
        ORDER BY P.BirthYear, P.Pid

        --BirthYear Pid
        --1910|G299001590
        --1910|X123400034
        --1912|10101013
        --1912|12345675
        --1912|86517510
        --1912|89550029
        --1912|G299001590
        --1913|G299001590
        --1915|G299001590
        --1916|86514204
        --1916|G299001590
        --1916|P120340419
    ```

-   在單一查詢中使用多個 CTE 定義
    ```sql
        WITH AA AS  
        (  
            SELECT 1 AS T1, 2 AS T2
        )
        , BB AS
        (  
            SELECT 4 AS T1, 5 AS T2

            UNION ALL  
          
            SELECT 2, 3  
        )

        SELECT * FROM AA
        UNION ALL
        SELECT * FROM BB
        
        --T1  T2
        --1    2
        --4    5
        --5    3
    ```

-   利用遞迴通用資料表運算式來顯示多層級的遞迴

    | EmployeeID | FirstName  | LastName | Title | DeptID | ManagerID |
    | ------ | ------ | ------ | ------ | ------ | ------ |
    | 1  | Ken  | Sánchez | Chief Executive Officer | 16 | NULL |
    | 16 | David | Bradley | Marketing Manager | 4 | 273 |
    | 23 | Mary | Gibson | Marketing Specialist | 4 | 16 |
    | 273 | Brian | Welcker | Vice President of Sales | 3 | 1 |
    | 274 | Stephen | Jiang | North American Sales Manager | 3 | 273 |
    | 275 | Michael | Blythe | Sales Representative | 3 | 274 |
    | 276 | Linda | Mitchell | Sales Representative | 3 | 274 |
    | 285 | Syed | Abbas | Pacific Sales Manager | 3 | 273 |
    | 286 | Lynn | Tsoflias | Sales Representative | 3 | 2 |

    ```sql
        WITH DirectReports(ManagerID, EmployeeID, Title, EmployeeLevel) AS   
        (  
            SELECT ManagerID, EmployeeID, Title, 0 AS EmployeeLevel  
            FROM dbo.MyEmployees   
            WHERE ManagerID IS NULL  
            UNION ALL  
            SELECT e.ManagerID, e.EmployeeID, e.Title, EmployeeLevel + 1  
            FROM dbo.MyEmployees AS e  
                INNER JOIN DirectReports AS d  
                ON e.ManagerID = d.EmployeeID   
        )  

        SELECT ManagerID, EmployeeID, Title, EmployeeLevel   
        FROM DirectReports  
        ORDER BY ManagerID

    --ManagerID	EmployeeID	Title	EmployeeLevel
    --NULL	1	Chief Executive Officer	0
    --1	273	Vice President of Sales	1
    --16	23	Marketing Specialist	3
    --273	16	Marketing Manager	2
    --273	274	North American Sales Manager	2
    --273	285	Pacific Sales Manager	2
    --274	275	Sales Representative	3
    --274	276	Sales Representative	3
    --285	286	Sales Representative	3
    ```

-   利用 MAXRECURSION 來取消陳述式
    -   可以利用 MAXRECURSION 來防止形式不良的遞迴 CTE 進入無限迴圈

    ```sql
        WITH cte (EmployeeID, ManagerID, Title) AS  
        (  
            SELECT EmployeeID, ManagerID, Title  
            FROM dbo.MyEmployees  
            WHERE ManagerID IS NOT NULL  
          UNION ALL  
            SELECT cte.EmployeeID, cte.ManagerID, cte.Title  
            FROM cte   
            JOIN  dbo.MyEmployees AS e   
                ON cte.ManagerID = e.EmployeeID  
        )  
        --Uses MAXRECURSION to limit the recursive levels to 2  
        SELECT EmployeeID, ManagerID, Title  
        FROM cte  
        OPTION (MAXRECURSION 2);
        
        --陳述式已結束。最大遞迴 2 已在陳述式完成之前用盡
    ```

# Merge

## 介紹
-   從與來源資料表聯結的結果，在目標資料表上執行插入、更新或刪除作業
-   當兩個資料表有複雜的比對的特性時，MERGE 陳述式的條件式行為表現最佳。 例如沒有資料列時插入資料列，或資料列相符時更新資料列。 只要根據另一個資料表的資料列更新資料表，就能以基本 INSERT、UPDATE 及 DELETE 陳述式來改善效能及延展性

## 語法
-   syntaxsql

    ```sql
        [ WITH <common_table_expression> [,...n] ]  
        MERGE
            [ TOP ( expression ) [ PERCENT ] ]
            [ INTO ] <target_table> [ WITH ( <merge_hint> ) ] [ [ AS ] table_alias ]  
            USING <table_source> [ [ AS ] table_alias ]
            ON <merge_search_condition>  
            [ WHEN MATCHED [ AND <clause_search_condition> ]  
                THEN <merge_matched> ] [ ...n ]  
            [ WHEN NOT MATCHED [ BY TARGET ] [ AND <clause_search_condition> ]  
                THEN <merge_not_matched> ]  
            [ WHEN NOT MATCHED BY SOURCE [ AND <clause_search_condition> ]  
                THEN <merge_matched> ] [ ...n ]  
            [ <output_clause> ]  
            [ OPTION ( <query_hint> [ ,...n ] ) ]
        ;  
          
        <target_table> ::=  
        {
            [ database_name . schema_name . | schema_name . ]  
          target_table  
        }  
          
        <merge_hint>::=  
        {  
            { [ <table_hint_limited> [ ,...n ] ]  
            [ [ , ] INDEX ( index_val [ ,...n ] ) ] }  
        }  

        <merge_search_condition> ::=  
            <search_condition>  
          
        <merge_matched>::=  
            { UPDATE SET <set_clause> | DELETE }  
          
        <merge_not_matched>::=  
        {  
            INSERT [ ( column_list ) ]
                { VALUES ( values_list )  
                | DEFAULT VALUES }  
        }  
          
        <clause_search_condition> ::=  
            <search_condition>
    ```

## 引數 (細節太多，請自行查閱官方文件)
-   WITH <common_table_expression>: 指定在 MERGE 陳述式範圍內定義的暫存具名結果集或檢視表
-   TOP ( expression ) [ PERCENT ]: 指定受到影響的資料列數目或百分比。  expression 可以是一個數字，也可以是資料列的百分比。 EX: TOP(1)、TOP(5)PERCENT
    -   I/O 效能可能會受到影響: MERGE 陳述式會針對來源和目標資料表執行完整資料表掃描
    -   可能產生不正確的結果: 請務必確定所有後續批次都是以新的資料列為目標，否則可能會發生不想要的行為
    -   若要確保結果正確:
        -   使用 ON 子句來判斷哪些來源資料列會影響現有的全新目標資料列
        -   在 WHEN MATCHED 子句中使用其他條件來判斷上一個批次是否已經更新目標資料列
-   ON <merge_search_condition>: 指定條件，在這些條件下，<table_source> 會與 target_table 聯結以決定其相符之處
    -   警告: 請務必只從目標資料表指定用於比對用途的資料行。請勿嘗試在 ON 子句中篩選出目標資料表的資料列 (例如指定 AND NOT target_table.column_x = value) 來改善查詢效能。這樣做可能會傳回非預期且不正確的結果。
-   table_hint_limited: 指定針對每個由 MERGE 陳述式所執行的插入、更新或刪除動作，套用到目標資料表的一或多個資料表提示。 WITH 關鍵字和括號都是必要的。
    -   不允許使用 NOLOCK 和 READUNCOMMITTED
    -   指定 INSERT 陳述式目標資料表之 TABLOCK 提示的效果，與指定 TABLOCKX 提示相同。 獨佔鎖定是在資料表上取得的。 當指定 FORCESEEK 時，該提示會套用到與來源資料表聯結之目標資料表的隱含執行個體
    -   警告: 使用 WHEN NOT MATCHED [ BY TARGET ] THEN INSERT 指定 READPAST 可能會導致違反 UNIQUE 條件約束的 INSERT 作業

## 備註
-   必須至少指定三個 MATCHED 子句中的一個，但可依任何順序指定這些子句。在同一個 MATCHED 子句中，不能更新變數一次以上。
-   在目標資料表上由 MERGE 陳述式所指定的任何插入、更新或刪除動作，都受限於資料表上定義的任何條件約束，包括任何串聯式參考完整性條件約束。 如果在目標資料表上將任何唯一索引的 IGNORE_DUP_KEY 設定為 ON，則 MERGE 會忽略此設定。
-   如果用在 MERGE 之後，@@ROWCOUNT (Transact-SQL) 會將插入、更新和刪除的資料列總數傳回用戶端。
-   MERGE 陳述式需要使用分號 (;) 做為陳述式結束字元。 若 MERGE 陳述式執行時缺少該結束字元，就會引發錯誤 10713。
-   使用佇列更新複寫時，不應該使用 MERGE 陳述式。 MERGE 與佇列更新觸發程序不相容。 請將 MERGE 陳述式取代成 Insert 或 Update 陳述式。

## MERGE 陳述式效能最佳化 (細節太多，請自行查閱官方文件)
-   索引最佳做法
-   JOIN 最佳做法
-   參數化最佳做法
-   TOP 子句最佳做法
-   大量載入最佳做法
-   衡量及診斷 MERGE 效能

## 範例
-   Upsert
    ```cs
        /// <summary>
        /// 更新增Nbs撥款資訊
        /// </summary>
        /// <param name="approvalNumber">批覆書編號</param>
        /// <param name="info">Nbs撥款資訊</param>
        /// <returns>異動筆數</returns>
        public int Upsert(string approvalNumber, NbsDrawdownInfo info)
        {
            if (info == null)
                return 0;

            var now = DateTime.Now;
            var infoJson = SerializeHelper.ToJsonString(info);

            var updateSql = $@"
                UPDATE [NCS].[NbsDrawdownInfo] SET
                     [DrawdownData] = @{nameof(infoJson)}
                    ,[CreatedOn] = @{nameof(now)}
                WHERE ApprovalNumber = @{nameof(approvalNumber)}";

            var insertSql = $@"
                INSERT INTO [NCS].[NbsDrawdownInfo]
                    ([ApprovalNumber]
                    ,[DrawdownData]
                    ,[CreatedOn])
                VALUES
                    (@{nameof(approvalNumber)}
                    ,@{nameof(infoJson)}
                    ,@{nameof(now)})";

            var count = 0;
            using (var conn = OpenNcs())
            {
                count = conn.Execute(updateSql, new { approvalNumber, infoJson, now });

                if (count == 0)
                    count = conn.Execute(insertSql, new { approvalNumber, infoJson, now });
            }

            return count;
        }
    ```
-   MERGE(Ods)
    ```cs
        /// <summary>
        /// 刷新信用卡資料們
        /// </summary>
        /// <param name="caseId">案件Id</param>
        /// <param name="infos">信用卡資料們</param>
        /// <param name="pid">要更新的Pid(infos的資料為此pid的資料)</param>
        /// <returns>刷新筆數</returns>
        public int Refresh(
            int caseId,
            List<CipInfo> infos,
            string pid)
        {
            if (caseId == 0)
                throw new ArgumentException($"{nameof(caseId)}不可為0");

            var oriIds = GetAllIds(caseId, pid);
            var now = Now;
            var news = GetOrNew(infos)
                .Where(x => x != null)
                .Select((x, i) => new Dm(x) {
                    Id = GetOrNew(oriIds).ElementAtOrDefault(i),
                    CaseId = caseId,
                })
                .ToList();

            var updateSql = $@"
                UPDATE [SbRe].[CipInfo]
                SET [IsDelete] = 1,
                    [ChangeOn] = @{nameof(now)}
                WHERE [CaseId] = @{nameof(caseId)}
                    AND [Pid] = @{nameof(pid)}";

            var mergeSql = $@"
                MERGE [SbRe].[CipInfo] AS T
                USING (VALUES
                    (@{nameof(Dm.Id)}
                    ,@{nameof(Dm.CaseId)}
                    ,@{nameof(Dm.Pid)}
                    ,@{nameof(Dm.WiseMemberRank)}
                    ,@{nameof(Dm.PibVersion)}
                    ,@{nameof(Dm.OtpServiceFlag)})
                ) AS C
                    ([Id]
                    ,[CaseId]
                    ,[Pid]
                    ,[WiseMemberRank]
                    ,[PibVersion]
                    ,[OtpServiceFlag])
                ON (T.[Id] = C.[Id])
                WHEN MATCHED THEN
                    UPDATE SET
                        [IsDelete] = 0,
                        [ChangeOn] = @{nameof(Dm.Time)},
                        [Pid] = C.[Pid],
                        [WiseMemberRank] = C.[WiseMemberRank],
                        [PibVersion] = C.[PibVersion],
                        [OtpServiceFlag] = C.[OtpServiceFlag]
                WHEN NOT MATCHED BY TARGET THEN
                    INSERT
                       ([CaseId]
                       ,[IsDelete]
                       ,[CreateOn]
                       ,[ChangeOn]
                       ,[Pid]
                       ,[WiseMemberRank]
                       ,[PibVersion]
                       ,[OtpServiceFlag])
                    VALUES
                        (C.[CaseId]
                        ,0
                        ,@{nameof(Dm.Time)}
                        ,@{nameof(Dm.Time)}
                        ,C.[Pid]
                        ,C.[WiseMemberRank]
                        ,C.[PibVersion]
                        ,C.[OtpServiceFlag]);";

            using (var conn = OpenNcs())
            {
                conn.Execute(updateSql, new { now, caseId, pid });
                return conn.Execute(mergeSql, news);
            }
        }
    ```




