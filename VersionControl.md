# Version Control

[Get Your Database Under Version Control](https://blog.codinghorror.com/get-your-database-under-version-control/) - Jeff Atwood
>You deploy the app, and you deploy the database. Like peanut butter and chocolate, they are two great tastes that taste great together.

  - Have a single and authoritative source
  - Generate a baseline schema
  - Merge branches
  - *Don't use shared database*

---

### 🐱‍👓 AP

Database Object 🔁 Project `Only For Development`

step 1. 開發人員同步 Repository (.sql)

  * Project Settings
  * User Permission & Role ❌
  * *Base Data*

step 2. Build Source Project

step 3. Merge Branches(`UAT`、`PROD`、*`Hotfix`*)

step 4. Build Target Project

---

### 🐱‍👤 Database Owner

Project ➡ Database Object

step 1. Compare Schema

step 2. Review SQL

 * SQL Guidelines

step 3. Generate Scripts
    
 * sqlcmd scripts to sql scripts

step 4. Test

### 📋 結構描述變更記錄

資料庫 -> 報表 -> 標準報表 -> 結構描述變更記錄
