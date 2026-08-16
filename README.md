# 🏫 W8 作業 — 用 Migration 建資料表

## 【任務情境】

前幾週我們都在「已經建立好的資料庫」上面操作，像是查詢、加索引來調整。這週把順序倒過來，輪到你來打地基：**從空的資料庫開始，建立資料表結構並寫入一些資料**。

撰寫過程中不會更動到應用程式邏輯（也不會寫到 API），而是專注在資料表結構的設計與建立。請運用第八週學習的 **EntitySchema（設計 entity）、Migration、Seeder** 等，來完成 Livefit 健身房與學校成績兩個情境的練習。


## 【環境準備】

1. 安裝並啟動 [Docker Desktop](https://www.docker.com/products/docker-desktop/)，並確認 Node.js 版本 >= 20（資料庫是運行在 Docker 裡，所以 Docker Desktop 記得要先打開）
2. **Fork** 作業專案 [https://github.com/hexschool/node-js-week8-2026](https://github.com/hexschool/node-js-week8-2026)，再 clone 同學自己的 fork
   - 這次需使用到 Fork，因為最終驗收是要跑自己 Repo 的 GitHub Actions
   - Fork 專案之後進到自己 Repo 的 **Actions** 分頁，按下「**I understand my workflows, go ahead and enable them**」按鈕，因為 GitHub 對於 fork 過來的專案預設會關閉 Actions，所以我們需手動啟用。
3. 這份作業裡有 `livefit/` 和 `school/` 兩個獨立的資料夾，各代表一個關卡。每一個關卡都要進入對應的資料夾，並且各跑一次下方的啟動（所以這樣的啟動總共會操作兩次）：

**第一關：LiveFit 健身房**

```bash
cd livefit
cp .env.example .env
npm install
npm start          # 透過 Docker 啟動一個全新的空資料庫，如果執行沒有問題，會在終端機看到：等資料庫就緒. ✅
```

**第二關：學校成績**

```bash
cd school          # 換到第二關的資料夾（若剛完成第一關、終端機位置會在 livefit/ 裡，記得先 cd .. 回到根目錄再輸入 cd school）
cp .env.example .env
npm install
npm start
```

（兩個關卡分別使用 5432、5433 port，所以兩邊的資料庫不會互相干擾）


## 【主線任務】

兩個關卡都是「設計 entity -> 使用 migration 建立資料表 -> 寫入一些資料」這套流程。建議順序：先完成第一關，再進到第二關。
**這週只需更動這幾個地方：**`entities/Xxxx.js`（自行新增）、`db/data-source.js`、`db/seed.js`。

其他檔案（`test/`、`scripts/`、`package.json`、`docker-compose.yml`、`.env.example`）都不可更動。⚠️ GitHub CI 會有一個檔案完整性的比對步驟，所以如果有更動到其他檔案，GitHub CI 會顯示 FAILED（不通過）

**第一關：LiveFit 健身房（需建立 3 張資料表）**

3 張資料表主鍵使用 `id`（uuid），表名 `tableName` 一律使用全大寫（例如 `USER`、`SKILL`）。

- **`USER`（教練）**
  - 欄位：`name` varchar(50) 必填、`email` varchar(320) 必填且唯一、`role` varchar(20) 必填、`created_at`、`updated_at`（建立／更新時間，由系統自動帶入）
  - 關聯：無
- **`SKILL`（技能）**
  - 欄位：`name` varchar(50) 必填且唯一
  - 關聯：無
- **`COURSE`（課程）**
  - 欄位：`name` varchar(100) 必填、`description` text 必填、`start_at` timestamp 必填、`end_at` timestamp 必填、`max_participants` integer 必填、`created_at`、`updated_at`（建立／更新時間，由系統自動帶入）
  - 關聯：`user_id` → `USER`（開課教練）、`skill_id` → `SKILL`（課程技能）


這關要寫入的資料：

- `SKILL`：3 筆（重訓、瑜珈、飛輪）
- `USER`：2 位教練（海格教練 `coach1@livefit.tw`、小美教練 `coach2@livefit.tw`，role 都是 `COACH`）
- `COURSE`：4 堂課（肌力入門班、週末飛輪、晨間瑜珈、核心特訓；每堂都要接上教練與技能）

加分挑戰（選做，建議先把主線做完，再回頭挑戰）：
- 情境：模擬實際開發很常遇到的狀況，就是資料表已經有資料，不過新的需求要在這個資料表新增一個欄位。
- 內容跟方法：在 `Course` 加上一個線上會議連結的新欄位（`meeting_url` varchar(2048)）。不可更動到舊的 migration，而是改好 Course entity 之後，再使用 `migration:generate` 生成新的 migration，最後執行 `migration:run`。
- 注意部分：這個新欄位需設置「可為空」，因為原先資料表已經有資料，如果這時加上一個 NOT NULL 欄位會導致 migration 失敗。

**第二關：學校成績（需建立 4 張資料表）**

4 張資料表主鍵使用 `id`（uuid），表名 `tableName` 一律使用全大寫（例如 `CLASS`、`SUBJECT`）。

- **`CLASS`（班級）**
  - 欄位：`name` varchar(50) 必填
  - 關聯：無
- **`SUBJECT`（科目）**
  - 欄位：`name` varchar(50) 必填
  - 關聯：無
- **`STUDENT`（學生）**
  - 欄位：`name` varchar(50) 必填
  - 關聯：`class_id` → `CLASS`（必填，一個學生屬於一個班級）
- **`GRADE`（成績）**
  - 欄位：`score` integer 必填
  - 關聯：`student_id` → `STUDENT`、`subject_id` → `SUBJECT`（皆必填，一筆成績會對應到一位學生與一個科目）

這關要寫入的資料：

- `CLASS`：至少 2 個班級
- `SUBJECT`：至少 2 個科目
- `STUDENT`：幾個學生
- `GRADE`：幾筆成績（要能 JOIN 回對應的學生與科目）

加分挑戰（選做，建議先把主線做完，再回頭挑戰）：
- 情境：和第一關的加分挑戰一樣，模擬資料表已經有資料，但新需求要在這個資料表再加一個欄位。
- 內容跟方法：在 `Grade` 加上一個補考分數的新欄位（`retake_score` integer）。不可更動到舊的 migration，而是改好 Grade entity 之後，再使用 `migration:generate` 生成新的 migration，最後執行 `migration:run`。
- 注意部分：這個新欄位需設置「可為空」，因為原先資料表已經有資料，如果這時加上一個 NOT NULL 欄位會導致 migration 失敗。


**關卡流程**

兩個關卡，都可依據下方的四個步驟：

**步驟一：撰寫 entities**
在 livefit/school 資料夾內，新增一個 `entities/` 資料夾，每個資料表使用一個檔案（.js），在這個檔案使用 `EntitySchema` 來建立欄位跟關聯。（撰寫關聯時要注意，`target` 是根據目標的 entity `name`，而不是 `tableName` 唷）

**步驟二：註冊 entity**
把寫好的 entity `require` 到 `db/data-source.js`，並將這些 entity 加進 `entities` 陣列。（如果有 entity 沒註冊到的話，下一個步驟的 `migration:generate` 就會看不到它）

**步驟三：使用 migration 建立資料表**

```bash
# 比對你的 entity 與資料庫目前的差異，把要建立資料表的 SQL 寫成一支 migration 檔案（這步只產生檔案，還沒有更動到資料庫）
# 後面的 db/migrations/Init 是輸出路徑跟檔名，實際會生成類似 db/migrations/<時間戳>-Init.js 的檔案
npm run migration:generate -- db/migrations/Init

# 建議先打開剛才生成的 migration 檔，確認裡面 CREATE TABLE 的數量是正確的（第一關 3 張、第二關 4 張）
# 執行 migration，把檔案裡的 SQL 執行到資料庫，資料表到這一步才真正被建立出來
npm run migration:run
```

**步驟四：寫入資料（補完 `db/seed.js`）**
`seed.js` 分成 `clearAll()`（先清空，確保重跑時資料不會疊加）和 `main()`（再寫入資料）兩個區塊；每一關把該撰寫的地方補好後，就在該關卡的資料夾執行 `npm run seed` 寫入資料。兩個關卡要撰寫的部分稍有不同：
- **第一關（livefit）**：`clearAll()` 清空功能已經寫好，只需根據 `main()` 裡的提示，把資料寫入即可。
- **第二關（school）**：`clearAll()` 的清空順序（`ORDER`）要自行按照外鍵的依賴順序填上，`main()` 則和第一關一樣，依提示把資料寫入。

## 【測試】

**作業繳交前必須通過**：
在各自的資料夾跑 `npm test`，✓ 表示通過、✕ 表示失敗。
測試內容有：有沒有建出資料表、欄位／型別對不對、外鍵有沒有接對、資料有沒有寫入。

- **第一關（livefit）**：共 13 條，全通過會看到 `Tests: 13 passed, 13 total`
- **第二關（school）**：共 11 條，全通過會看到 `Tests: 11 passed, 11 total`

本機兩個關卡都通過後，把作業 push 到 GitHub。GitHub Actions 會用一個全新的資料庫，重新跑一次你寫的 migration 和 seeder，然後再跑測試，兩個關卡的測試會各跑一次，兩邊都通過才算完成。


## 【常見問題】

**Q：`npm start` 出現「等了 60 秒資料庫還沒就緒...」？**
A：先確認 Docker Desktop 有開著（資料庫是跑在 Docker 裡的）。如果確定有開、但還是卡住，可以執行 `docker compose logs postgres` 看資料庫的啟動訊息，通常就能看出原因。

**Q：5432 或 5433 這個 port 被電腦上其他程式佔用？**
A：
承上題，如果 Docker 有開啟但 `npm start` 還是失敗，有個常見原因是「這個 port 已經被電腦上其他程式佔用」（最常見的是本機自己裝的 PostgreSQL）。可以這樣處理：

調整方向一，把那一關 `.env` 裡的 `DB_PORT` 改成其他號碼（例如第一關改 5434、第二關改 5435），再執行 `npm run db:reset` 重新啟動。compose 和所有腳本都會去讀這個值。

調整方向二，如果能確定佔用的那個程式不重要、停掉也不會影響系統運作，也可以直接把它停用，把 port 空出來。

**Q：第二關的分數（score）欄位該用什麼型別？**
A：可使用 `integer`。記得不要用 `numeric`，因為 `numeric` 經過 node-pg 取出來會變成字串，之後要拿來計算時還得自己轉型，反而容易出錯。

**Q：`migration:generate` 報錯，說找不到某個 entity 的 metadata（No metadata for ... was found）？**
A：多半是關聯的 `target` 填成了資料表名稱。`target` 要填「目標 entity 的 `name`」，不是全大寫的 `tableName`。

**Q：`migration:generate` 顯示 No changes，或終端機跳出一片紅色的錯誤訊息？**
A：這代表你的 entity 和資料庫目前沒有差異，TypeORM 沒有變更可以生成。最常見的原因是 entity 寫好了，卻忘了在 `db/data-source.js` 的 `entities` 陣列註冊它；沒有註冊的 entity，`migration:generate` 是看不到的。

**Q：執行 `npm run seed` 時報外鍵（FK）錯誤？**
A：寫入資料的順序要對：先寫入「被指著的表」，再寫入「指著別人的表」。以第一關來說，先寫入 USER／SKILL，最後才寫入 COURSE；第二關則是先寫入 CLASS／SUBJECT／STUDENT，最後才寫入 GRADE。清空的順序則剛好相反。

**Q：在 DBeaver / psql 使用 SQL 查詢資料表，卻顯示找不到資料表，或出現語法錯誤？**
A：這些資料表都是用全大寫建立的（TypeORM 會加上雙引號、區分大小寫）。而 PostgreSQL 中沒加引號的名稱會被轉成小寫，所以 `SELECT * FROM CLASS` 其實是去找 `class` 資料表（依照這份作業就會找不到）。因此在查詢這份作業相關的資料表時，需要加上雙引號並維持大寫，例如 `SELECT * FROM "CLASS"`、`SELECT * FROM "USER"`。（`USER` 剛好是 PostgreSQL 的保留字，不加引號會直接語法錯誤，更要記得加上引號。）

**Q：想把資料庫清空、整個重來？**
A：執行 `npm run db:reset`，資料庫會重建成空的，接著重跑 `migration:run` 和 `seed` 即可（migration 檔還在，不用再 generate）。

**Q：push 前要記得推哪些檔案？**
A：除了 entity，`db/migrations/` 裡生成好的 migration 檔案也要記得推上去。因為 GitHub CI 只會執行 `migration:run`（不會重新 `generate`），所以如果只推了 entity，然後忘記推 migration 的話，CI 會因為沒有資料表可建立而失敗。

**Q：本機測試都過了，但 GitHub Actions（CI）卻沒通過？**
A：通常是兩個原因。① 忘了把 migration 檔一起推上去（詳細可看上一個 QA）。② 不小心改到了不可更動的檔案（`test/`、`scripts/`、`package.json`、`docker-compose.yml`、`.env.example`）；CI 過程會比對這些檔案的 checksum，只要動過就會直接失敗。


## 小筆記

Week 8：Express + TypeORM 整合 — Migration 與 Seeding
https://hackmd.io/@hexschool/BJloatPHfe

第八堂主線任務
https://chalk-freedom-ec6.notion.site/3916ab47eb4880228161cbc56c0140a5