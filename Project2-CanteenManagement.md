# PROJECT_2: CANTEEN MANAGEMENT SYSTEM

## 系統設計 Introduction
> 是一套為學校餐廳打造的**線上訂餐與營運管理平台**，
> 目標是把「排隊點餐、現金找贖、廚房靠紙單／口頭」的傳統流程，轉為**線上預訂 + 電子錢包支付 + 系統出餐看板**的完整自動流程。
>

## 1. 要解決的問題 Problem

| 痛點 Pain Point | 系統對策 Solution |
|---|---|
| 午膳時段人潮擠塞、排隊時間長 | 線上**預先下單**，過了截止時間自動順延隔日（pre-order） |
| 現金收找費時、易出錯 | **電子錢包**（Wallet）+ Stripe 線上增值，下單即扣款 |
| 廚房靠紙單／口頭傳遞，容易漏單 | **Kitchen Board** 三欄，15 秒自動更新 |
| 顧客不知道餐點完成時間 | **訂單追蹤**時間軸，10 秒輪詢即時狀態 |
| 餐點賣完仍被下單、庫存靠人手記錄 | 下單時**即時扣減庫存**，管理端庫存警示（≤5） |
| 缺乏營運數據 | **Dashboard**：營收、訂單量、熱門餐點圖表 |

## 2. 核心功能 Features

### 顧客 Customer
- 瀏覽菜單（依 Menu 分頁，支援「All」匯總）
- 購物車（存 `localStorage`，重新整理不遺失）
- 下單並選擇支付方式（錢包 / 現金）
- 即時訂單追蹤（10 秒輪詢狀態時間軸）
- 歷史訂單查詢與取消
- 電子錢包：查看餘額、Stripe 線上增值、交易明細
- 個人資料與密碼修改

### 廚房 Kitchen
- 三欄看板（已下單 / 準備中 / 可取餐），15 秒自動更新
- 一鍵推進訂單狀態：接單 → 準備 → 可取餐 → 完成取餐
- 拒絕訂單（自動回補庫存並退款）

### 管理後台 Admin
- **儀表板**：訂單數、用戶數、營收、熱門餐點圖表、營收與本週訂單圖表
- **訂單管理**：全部訂單列表、狀態篩選／搜尋、直接改狀態
- **餐點管理**：Item CRUD、庫存警示（≤5 提示補貨）、與 Menu 關聯
- **菜單管理**：Menu CRUD、啟用／停用
- **狀態日誌**：完整的訂單狀態異動紀錄
- **系統設定**：訂餐時間窗設定、建立／刪除廚房帳號

## 3. 系統概觀 Flow
Sequence Login (JWT/Bearer Token/Google Login)
> <img width="3328" height="4216" alt="mermaid-diagram-2026-07-27-013824" src="https://github.com/user-attachments/assets/7e46eaf0-3983-4907-84b7-07bc51feb233" />

Sequence Ordering (以**顧客瀏覽菜單**透過各層的路徑讀取請求)
> <img width="3004" height="4014" alt="mermaid-diagram-2026-07-27-013524" src="https://github.com/user-attachments/assets/9d47fa1f-1c4d-4ab0-88a3-b7d7f87ce3a7" />

## 4. 系統架構 Architecture

- **前後端分離**（Decoupled SPA-less Frontend + REST API）— 兩個獨立 repo、獨立部署
- **分層架構**（Layered Architecture）— Controller → Service → Repository → Entity
- **介面／實作分離** — 每個 Controller 與 Service 皆有 interface（`Impl` 與 `*Operation`）
- **Stateless 認證** — 無伺服器端 Session，可直接水平擴展

| # | 元件 Component | 技術 Tech | Port | 職責 Responsibility |
|---|---|---|---|---|
| 1 | **Frontend Static Site** | HTML / CSS / JavaScript | `443` | UI 呈現、購物車暫存、頁面層角色、閒置登出計時 |
| 2 | **Backend REST API** | Java / Spring Boot | `8080`（`${PORT}`） | 業務邏輯、認證授權、交易控制、外部金流整合 |
| 3 | **Relational Database** | PostgreSQL | `5432` | 持久化資料、CHECK 約束、Row-level Lock|
| 4 | **Stripe Checkout** | Stripe Java | `443`（外部 HTTPS） | 錢包增值付款頁、付款狀態查證 |
| 5 | **Google Identity** | GIS + `oauth2-jose` | `443`（外部 HTTPS） | 簽發 ID Token、提供 JWKS 供後端驗簽 |

## 5. 技術堆疊（Tech Stack）

- **Spring Boot 4.0.6 / Java 21** — 單一後端服務
- **Spring Web MVC** — REST controller，介面（`*Operation`）與實作（`Impl`）分離
- **Spring Data JPA + Hibernate 7.2** — Entity 持久化；
- **Spring Security** — Filter Chain、集中式 URL 授權、BCrypt 密碼雜湊；
- **spring-security-oauth2-jose** — 以 Google JWKS 驗證 ID Token 的簽章 
- **PostgreSQL** — 系統的資料真實來源 (Railway)
- **Stripe Java SDK 32.1.0** — Checkout Session 建立與查證，**僅用於錢包增值**
- **Google Identity Services (GIS)** — 社交登入，透過 Client ID
- **Lombok / Maven / JUnit 5 + Mockito** — 單元測試
- **前端狀態** — `sessionStorage`（登入 session）+ `localStorage`（購物車）+ `api.js` Bearer Token
- **部署** — Railway（後端 PaaS，機密全由環境變數注入）

