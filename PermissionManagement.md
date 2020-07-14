# Permission Management


### [🔍](https://docs.microsoft.com/zh-tw/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver15) 資料庫層級角色

>為了輕鬆管理資料庫中的權限， SQL Server 提供了幾個 「角色」 (Role)，這些角色是分組其他主體的安全性主體。 它們就像是 Windows 作業系統中的 群組 Microsoft 。 資料庫層級角色的權限範圍為整個資料庫。

#### ⚔ 固定資料庫角色

  - `db_owner` 固定資料庫角色的成員可以在資料庫上執行所有的組態和維護活動，也可以在 SQL Server中卸除資料庫。 (在 SQL Database 和 SQL 資料倉儲中，某些維護活動需要伺服器層級的權限，而且無法由 db_owners執行。)
  - db_securityadmin 固定資料庫角色的成員可以修改角色成員資格 (僅自訂角色) 以及管理權限。 此角色的成員可能會提升其權限，因此其動作應受到監視。
  - db_accessadmin 固定資料庫角色的成員可以針對 Windows 登入、Windows 群組及 SQL Server 登入加入或移除資料庫的存取權。
  - db_backupoperator 固定資料庫角色的成員可以備份資料庫。
  - `db_ddladmin` 固定資料庫角色的成員可在資料庫中執行任何「資料定義語言」(DDL) 的命令。
  - `db_datawriter` 固定資料庫角色的成員可以加入、刪除或變更所有使用者資料表中的資料。
  - `db_datareader` 固定資料庫角色的成員可以從所有使用者資料表讀取所有資料。
  - db_denydatawriter 固定資料庫角色的成員不能加入、修改或刪除資料庫中使用者資料表的任何資料。
  - db_denydatareader 固定資料庫角色的成員不能讀取資料庫中使用者資料表的任何資料。

#### ⚔ 使用者定義資料庫角色

##### 🐱‍👤 [Role_AP_Advanced]

        UAT

          * READ
          * WRITE
          * ALTER
          * EXECUTE

        SIT

          * OWNER

##### 🐱‍👓 [Role_AP]

        UAT

          * READ

        SIT

          * READ
          * WRITE
          * ALTER
          * EXECUTE

##### User Default Schema: `dbo`
