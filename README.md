# 1.Stock Map System — 系統結構設計

> 一個US股票市場 **heatmap（熱力圖）+ candlestick（K 線圖）** 的網頁應用程式
> 系統會從外部資料供應商收集即時報價、公司基本資料以及 OHLCV 歷史資料，
> 將資料儲存於 PostgreSQL，並以 Redis 快取即時報價與公司資料加速讀取，
> 最後以一個會自動更新的 heatmap 與 candlestick 圖形介面呈現。
>
> <img width="1917" height="967" alt="image" src="https://github.com/user-attachments/assets/eda0ff01-6237-4285-84f1-bc8222339bb7" />

<img width="1917" height="955" alt="image" src="https://github.com/user-attachments/assets/11dd3e54-b848-4547-b46b-1d8b9d2bba80" />

## 2. 系統概觀

整個系統由 **兩個 Spring Boot 專案**、一個 **純靜態前端**、一組 **Python ETL 腳本**、
兩個 **資料儲存（data store）** 組成，並與三個 **外部資料供應商（external provider）** 互動。

<img width="3783" height="2454" alt="mermaid-diagram-2026-07-24-042712" src="https://github.com/user-attachments/assets/333a612d-ca54-4bc6-953c-911376774a26" />

## 3. 技術堆疊（Technology Stack）

- **Spring Boot 4.1 / Java 21** — 兩個後端服務
- **Spring Web MVC** — REST controller（兩個服務使用）
- **Spring Data JPA + Hibernate** — stock-data-app 的持久化（persistence）
- **Spring Data Redis** — 即時報價（手動 `RedisTemplate`）+ 公司資料（`@Cacheable`）
- **Spring Scheduler** — 每日公司資料刷新、即時報價輪詢
- **PostgreSQL** — 系統的資料真實來源（生產環境 Supabase 託管）
- **Redis** — 即時報價 + 公司資料快取
- **Python（pandas + SQLAlchemy）** — ETL 腳本（Extract、Transform、Load）
- **前端繪圖** — **d3-hierarchy** **TradingView Lightweight Charts**（treemap 版面 + candlestick）；
  HTML / CSS / JS 分檔，純靜態、無 build step
- **Docker Compose** — Localhost一次Run測試
- **部署** — AWS EC2（systemd + 本機 Redis）、Supabase（PostgreSQL）、Vercel（前端）、
  GitHub Actions（CI/CD）
