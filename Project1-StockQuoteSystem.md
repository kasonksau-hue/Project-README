# PROJECT_1: STOCK QUOTE SYSTEM 

## 系統設計 Introduction
> 一個US股票市場 **Heatmap（熱力圖）+ Candlestick（K 線圖）** 的網頁應用程式涵蓋整個SDLC - 
> 系統會從外部資料供應商收集即時報價、公司基本資料以及 OHLCV 歷史資料，
> 將資料儲存於 PostgreSQL，並以 Redis 快取即時報價與公司資料加速讀取，
> 最後以一個會自動更新的 Heatmap 與 Candlestick 圖形介面呈現。
>
> 
## 1. 專案簡介 Overview

本平台以互動式 Treemap 呈現 S&P 500 市值 **Top 100** 股票的當日表現：

- **方塊面積大小（tile Area）** 對應市值（Market capitalization）
- **方塊顏色深淺（tile Color）** 對應當日漲跌幅（Daily % change，綠紅升跌感知色階）
- **游標懸停（Hover）** 即時顯示迷你走勢預覽卡（Preview sparkline）
- **點擊方塊** 顯示該股 Candlestick K 線圖與公司基本資料
- **響應式設計** — 5 Typography Hierarchy 適配 Desktop 至 Mobile device.

資料每約 1.2 秒自動刷新（`setInterval`），以反映即時報價變化。
> <img width="1917" height="967" alt="image" src="https://github.com/user-attachments/assets/eda0ff01-6237-4285-84f1-bc8222339bb7" />

> <img width="1917" height="955" alt="image" src="https://github.com/user-attachments/assets/11dd3e54-b848-4547-b46b-1d8b9d2bba80" />
https://kason-stock-headmap.vercel.app/

## 2. 系統概觀 Flow

整個系統由 **兩個 Spring Boot 專案**、一個 **純靜態前端**、一組 **Python ETL 腳本**、
兩個 **資料儲存（data store）** 組成，並與三個 **外部資料供應商（external provider）** 互動。

Sequence Read
> <img width="2928" height="2728" alt="mermaid-diagram-2026-07-26-175139" src="https://github.com/user-attachments/assets/a4c002ba-fbf8-4d76-8679-3a128bfc38c3" />
Sequence Maintenance
> <img width="4271" height="2386" alt="mermaid-diagram-2026-07-26-181810" src="https://github.com/user-attachments/assets/899e394e-c1ec-4afb-b3d4-967601bab278" />

## 3. 系統架構 Architecture

| 元件 Component | 技術 Tech | Port | 職責 Responsibility |
|-----------|------|------|----------------|
| **frontend** | HTML / CSS / JavaScript、d3-hierarchy、Lightweight Charts | static | Treemap heatmap 與 candlestick UI，non building steps |
| **stock-data** | Spring Boot、Java、JPA、Redis | 8081 | 系統資料來源（system of record）——資料模型、排程、運算、前端 `/api/*`（及內部 `/data/*`）|
| **data-provider** | Spring Boot、Java | 8082 | 無狀態（stateless），封裝 Finnhub API and token 安全性 |
| **PostgreSQL** | Supabase（managed）| 5432 | Save 股票（symbols）、公司 profiles、歷史日線 OHLC |
| **Redis** | AWS EC2 localhost | 6379 | 即時報價與公司資料快取（兩種快取策略，non-relational）|
| **python-etl** | Python、pandas、SQLAlchemy | — | 由外部 API 載入 symbols；Cron job每日增量載入 OHLC |

## 4. 技術堆疊（Tech Stack）

- **Spring Boot 4.1 / Java 21** — 兩個後端服務
- **Spring Web MVC** — REST controller（兩個服務使用）
- **Spring Data JPA + Hibernate** — stock-data-app Entity 持久化
- **Spring Data Redis** — 即時報價（手動 `RedisTemplate`）+ 公司資料（`@Cacheable`）
- **Spring Scheduler** — 每日公司資料刷新、即時報價輪詢
- **PostgreSQL** — 系統的資料真實來源（生產環境 Supabase 託管）
- **Redis** — 即時報價 + 公司資料快取
- **Python（pandas + SQLAlchemy）** — ETL 腳本（Extract、Transform、Load）
- **前端繪圖** — **d3-hierarchy** and **TradingView Lightweight Charts**（treemap 版面 + candlestick）；
  HTML / CSS / JS 分檔，純靜態、non building steps 
- **Docker Compose** — Localhost一次Run測試
- **部署** — AWS EC2（systemd + localhost Redis）、Supabase（PostgreSQL）、Vercel（前端）、
  GitHub Actions（CI/CD）
