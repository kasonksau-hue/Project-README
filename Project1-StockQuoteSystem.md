# PROJECT_1: STOCK MARKET INTELLIGENCE PLATFORM

## 系統設計 Introduction
> 一個US股票市場 **Heatmap（熱力圖）+ Candlestick（K 線圖）+ AI Assistant（分析助手）** 的網頁應用程式，涵蓋整個SDLC -
> 個人專案，目標是在**成本限制的 Infrastructure**下，模擬一套接近生產環境水準的即時股票資料系統。
> 系統會從外部資料供應商收集即時報價、公司基本資料以及 OHLCV 歷史資料，
> 將資料儲存於 PostgreSQL，並以 Redis 快取即時報價與公司資料加速讀取，
> 最後以一個會自動更新的 Heatmap、Candlestick 圖形介面，以及一個能理解當下市場數據的 AI 聊天助手呈現。
>
> 
## 1. 專案簡介 Overview

本平台以互動式 Treemap 呈現 S&P 500 市值 **Top 100** 股票的當日表現：

- **方塊面積大小（tile Area）** 對應市值（Market capitalization）
- **方塊顏色深淺（tile Color）** 對應當日漲跌幅（Daily % change，綠紅升跌感知色階）
- **游標懸停（Hover）** 即時顯示迷你走勢預覽卡（Preview sparkline）
- **點擊方塊** 顯示該股 Candlestick K 線圖與公司基本資料
- **AI 聊天助手** — 內建專業分析師問答功能，使用者可針對系統數據提問，以及觸發外部資料搜尋
  補足資訊。
- **響應式設計** — 5 Typography Hierarchy 適配 Desktop 至 Mobile device.

資料每約 1.2 秒自動刷新（`setInterval`），以反映即時報價變化。
> <img width="1918" height="967" alt="image" src="https://github.com/user-attachments/assets/fdbb14b7-4e26-4778-9695-97d4feab538c" />

> <img width="1906" height="963" alt="image" src="https://github.com/user-attachments/assets/75b3b369-661d-41e5-931a-7cabc8798dc0" />

https://kason-stock-headmap.vercel.app/

## 2. 系統概觀 Flow

整個系統由 **兩個 Spring Boot 專案**、一個 **純靜態前端**、一組 **Python ETL 腳本**、
兩個 **資料儲存（Data Store）** 組成，並與四個 **外部資料供應商（包括Google Gemini API and External Data APIs）互動**

Sequence Read
> <img width="2928" height="2728" alt="mermaid-diagram-2026-07-26-175139" src="https://github.com/user-attachments/assets/a4c002ba-fbf8-4d76-8679-3a128bfc38c3" />
Sequence Maintenance
> <img width="4271" height="2386" alt="mermaid-diagram-2026-07-26-181810" src="https://github.com/user-attachments/assets/899e394e-c1ec-4afb-b3d4-967601bab278" />

Sequence - AI Assistant
- User Browser → Ask（question / history）→  Mode determined by rule-based classifier.
- Fast mode: Internal market data assembled from database system.
- Think mode: Retrieve supplementary external sources to cover topics outside internal coverage. 
- Structured prompt deliver to LLM for reasoning.
- Generate a response governed by system instructions.

### 需求 → 設計決策 Requirements-driven trade-offs
(每一個重要的設計決策都由特定的系統約束或需求驅動，而非隨意選擇)

| 限制 Constraint | 設計決策 Decision | 取捨 Trade-off |
|---|---|---|
| Free-tier API 限流（60 calls/min） | Round-robin `@Scheduled` job，每 1.2 秒輪詢一檔（約 50 calls/min） | 用「更新頻率」換「限流穩定性」——犧牲極致即時性，換取成本控制 |
| EC2 free tier 僅 1GB RAM | AI 對話歷史後端截斷、IP rate limit、快取 TTL 控管 | 犧牲部分使用彈性（無法無限對話），換取服務不會因單一使用者而受影響/癱瘓 |
| 第三方 LLM 額度/穩定性不保證 | Gemini 呼叫失敗自動降級（從think mode轉為fast mode重試） | 犧牲該次回答的深度，換取功能不會 100% 失敗 |
| AI 若給具體買賣建議 → 潛在合規/法律風險 | System Prompt 明確限制：不給標的/目標價/進出場時機 | 犧牲「更像真人顧問」的體驗，換取產品定位在安全的合規邊界內 |

## 3. 系統架構 Architecture

| 元件 Component | 技術 Tech | Port | 職責 Responsibility |
|-----------|------|------|----------------|
| **frontend** | HTML / CSS / JavaScript、d3-hierarchy、Lightweight Charts | static | Treemap heatmap, candlestick and AI 分析助手 UI|
| **stock-data** | Spring Boot、Java、JPA、Redis | 8081 | 系統資料來源 -資料模型、排程、運算、前端 `/api/*` 及內部 `/data/*`|
| **data-provider** | Spring Boot、Java | 8082 | Stateless，External API integration and token security |
| **PostgreSQL** | Supabase（managed）| 5432 | Persistent Stock data、公司 profiles、歷史日線 OHLC |
| **Redis** | AWS EC2 localhost | 6379 | 即時報價cache 與 公司資料cache (rate limiting) |
| **python-etl** | Python、pandas、SQLAlchemy | — | 由外部 API 載入 Symbols；Daily OHLC data pipeline |

## 4. 技術堆疊（Tech Stack）

- **Spring Boot 4.1 / Java 21** — 兩個後端服務
- **Spring Web MVC** — REST controller（兩個服務使用）
- **Spring Data JPA + Hibernate** — stock-data-app Entity 持久化
- **Spring Data Redis** — 即時報價（手動 `RedisTemplate`）+ 公司資料（`@Cacheable`）+ AI對話快取
- **Spring Scheduler** — 每日公司資料刷新、即時報價輪詢
- **PostgreSQL** — 系統的資料真實來源（生產環境 Supabase 託管）
- **Redis** — 即時報價 + 公司資料快取
- **Google Gemini API** — AI 聊天助手核心 System Prompt 設計
- **Python（pandas + SQLAlchemy）** — ETL 腳本（Extract、Transform、Load）
- **前端繪圖** — **d3-hierarchy** and **TradingView Lightweight Charts**（treemap 版面 + candlestick）；
  HTML / CSS / JS 分檔，純靜態、non building steps 
- **Docker Compose** — Localhost一次Run測試
- **部署** — AWS EC2（systemd + localhost Redis）、Supabase（PostgreSQL）、Vercel（前端）、
  GitHub Actions（CI/CD）
