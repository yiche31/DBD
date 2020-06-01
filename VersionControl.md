# Version Control

[Get Your Database Under Version Control](https://blog.codinghorror.com/get-your-database-under-version-control/) - Jeff Atwood
>You deploy the app, and you deploy the database. Like peanut butter and chocolate, they are two great tastes that taste great together.

  - Have a single and authoritative source
  - Generate a baseline schema
  - Merge branches
  - *Don't use shared database*

---

### 🐱‍👓 AP

Database 🔁 Project `only for development`

step1 開發人員同步 Repository

  * User Permission ❌

step2 Merge Branches(`UAT`、`PROD`、*`Hotfix`*)

step3 Build Project

---

### 🐱‍👤 Database Owner

Project ➡ Database

step1 Compare Schema

step2 Review SQL

  * SQL Guidelines

step3 Generate Scripts(SQLCMD)

step4 Test



### 📋 結構描述變更記錄

資料庫 -> 報表 -> 標準報表 -> 結構描述變更記錄