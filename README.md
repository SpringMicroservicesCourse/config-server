# Spring Cloud Config Server ⚡

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.5-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2024.0.2-blue.svg)](https://spring.io/projects/spring-cloud)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 專案介紹

這是一個基於 Spring Cloud Config Server 的**集中式配置管理服務**，提供微服務架構中各個服務的統一配置管理功能。透過 HTTP API 為各個微服務提供外置配置，支援多種後端儲存方式，包括 Git、SVN、本地檔案系統等。

### 核心功能
- **集中式配置管理**：統一管理所有微服務的配置檔案
- **多環境支援**：支援開發、測試、生產等不同環境的配置管理
- **動態配置更新**：支援配置的動態刷新，無需重啟服務
- **版本控制整合**：使用 Git 作為配置儲存後端，享受版本控制的所有優勢
- **服務註冊發現**：整合 Consul 進行服務註冊與發現

> 💡 **為什麼選擇 Spring Cloud Config Server？**
> - 提供 RESTful API 進行配置存取，整合簡單
> - 支援多種配置儲存後端，靈活性高
> - 與 Spring Boot 生態系統完美整合
> - 支援配置加密，保護敏感資訊安全

### 🎯 專案特色

- **Git 後端整合**：使用 Git 倉庫作為配置儲存，享受版本控制優勢
- **多環境配置**：透過 Profile 機制支援不同環境的配置管理  
- **服務發現整合**：與 Consul 整合，提供服務註冊與發現功能
- **健康檢查**：內建 Actuator 端點，提供應用程式健康狀態監控

## 技術棧

### 核心框架
- **Spring Boot 3.4.5** - 微服務基礎框架，提供自動配置和快速開發能力
- **Spring Cloud Config Server** - 分散式配置管理核心元件
- **Spring Cloud Consul Discovery** - 基於 Consul 的服務發現機制

### 開發工具與輔助
- **Maven** - 專案建置與依賴管理工具
- **Git** - 配置檔案版本控制系統
- **Consul** - 服務註冊與發現中心
- **Actuator** - 應用程式監控與管理端點

## 專案結構

```
config-server/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── tw/fengqing/spring/cloud/configserver/
│   │   │       └── ConfigServerApplication.java    # 主要啟動類別
│   │   └── resources/
│   │       ├── application.properties              # 應用程式配置檔案
│   │       └── bootstrap.properties               # 啟動配置檔案  
│   └── test/
│       └── java/
│           └── tw/fengqing/spring/cloud/configserver/
│               └── ConfigServerApplicationTests.java
├── pom.xml                                        # Maven 專案配置檔案
└── README.md                                      # 專案說明文件
```

## 快速開始

### 前置需求
- **Java 21** 或更高版本
- **Maven 3.6+** 用於專案建置
- **Git** 用於配置檔案版本控制
- **Consul** 作為服務註冊中心（選用）

### 安裝與執行

1. **克隆此倉庫：**
```bash
git clone https://github.com/SpringMicroservicesCourse/spring-cloud-config-server.git
```

2. **進入專案目錄：**
```bash
cd config-server
```

3. **準備配置倉庫：**
```bash
# 建立 Git 配置倉庫
mkdir -p ~/fengqing-spring-cloud-config-git-repo
cd ~/fengqing-spring-cloud-config-git-repo
git init

# 建立範例1配置檔案
cat > waiter-service.yml <<'YAML'
order:
  discount: 80
  waiterPrefix: "fengqing-"
YAML

git add .
git commit -m "init"

# 建立範例2配置檔案
cat > waiter-service-dev.yml <<'YAML'
order:
  discount: 60
YAML

git add  waiter-service-dev.yml
git commit -m ""
```

4. **編譯專案：**
```bash
mvn clean compile
```

5. **執行應用程式：**
```bash
mvn spring-boot:run
```

6. **驗證服務運行：**
```bash
# 檢查健康狀態
curl http://localhost:8888/actuator/health

# 取得配置檔案
curl http://localhost:8888/waiter-service.yml
```

## 進階說明

### 環境變數
```properties
# Git 倉庫配置
CONFIG_GIT_URI=file:///path/to/your/config-repo

# Consul 連線設定
CONSUL_HOST=localhost
CONSUL_PORT=8500

# 應用程式設定
SERVER_PORT=8888
LOG_LEVEL=INFO
```

### 主要配置檔案說明

#### application.properties
```properties
# 服務器埠號設定
server.port=8888

# 管理端點設定 - 開放所有監控端點
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

# Consul 服務註冊設定
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.prefer-ip-address=true

# Git 倉庫配置 - 指定配置檔案儲存位置
spring.cloud.config.server.git.uri=file:///Users/aaron/Documents/SynologyDrive/VAULT/200-AREA/arch/coding/java/spring/fengqing-spring-cloud-config-git-repo
```

### 配置檔案命名規則

Config Server 支援以下配置檔案命名格式：

| 格式 | 說明 | 範例 |
|------|------|------|
| `{application}.yml` | 應用程式預設配置 | `waiter-service.yml` |
| `{application}-{profile}.yml` | 特定環境配置 | `waiter-service-dev.yml` |
| `application.yml` | 全域共用配置 | `application.yml` |

### API 端點說明

| HTTP Method | 端點 | 說明 |
|-------------|------|------|
| GET | `/{application}/{profile}` | 取得JSON格式配置 |
| GET | `/{application}.yml` | 取得YAML格式配置 |
| GET | `/{application}-{profile}.yml` | 取得特定環境YAML配置 |
| GET | `/actuator/health` | 健康檢查端點 |

### 使用範例

```bash
# 取得 waiter-service 的預設配置
curl http://localhost:8888/waiter-service.yml

# 取得 waiter-service 開發環境配置
curl http://localhost:8888/waiter-service-dev.yml

# 取得 JSON 格式配置
curl http://localhost:8888/waiter-service/default

# 取得特定環境的 JSON 配置
curl http://localhost:8888/waiter-service/dev
```

## 參考資源

- [Spring Cloud Config 官方文件](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/)
- [Spring Boot Actuator 指南](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Consul 服務發現](https://www.consul.io/docs/discovery)

## 注意事項與最佳實踐

### ⚠️ 重要提醒

| 項目 | 說明 | 建議做法 |
|------|------|----------|
| 配置安全性 | 敏感資訊保護 | 使用 Spring Cloud Config 加密功能 |
| Git 倉庫權限 | 配置檔案存取控制 | 設定適當的 Git 倉庫存取權限 |
| 網路安全 | Config Server 暴露端點 | 在生產環境中限制網路存取 |
| 效能考量 | Git 倉庫同步 | 定期清理 Git 歷史，避免倉庫過大 |

### 🔒 最佳實踐指南

- **配置分層管理**：使用 `application.yml` 存放通用配置，個別服務配置檔案存放特定設定
- **環境隔離**：透過 Profile 機制明確區分不同環境的配置
- **安全考量**：敏感資訊如資料庫密碼、API 金鑰等應使用加密功能保護
- **版本控制**：充分利用 Git 的分支與標籤功能管理配置版本
- **監控告警**：善用 Actuator 端點監控 Config Server 的健康狀態
- **災難復原**：定期備份 Git 配置倉庫，確保配置資料安全

### 🚀 開發團隊協作建議

- **配置變更流程**：建立標準的配置變更 Pull Request 流程
- **環境一致性**：確保各環境配置結構一致，只有數值不同
- **文件維護**：在配置檔案中添加清楚註解，說明各項設定的用途 [[memory:2558636]]
- **測試驗證**：配置變更前先在測試環境驗證，確保不影響服務運行

## 授權說明

本專案採用 MIT 授權條款，詳見 LICENSE 檔案。

## 關於我們

我們主要專注在敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。喜歡把先進技術和實務經驗結合，打造好用又靈活的軟體解決方案。

## 聯繫我們

- **FB 粉絲頁**：[風清雲談 | Facebook](https://www.facebook.com/profile.php?id=61576838896062)
- **LinkedIn**：[linkedin.com/in/chu-kuo-lung](https://www.linkedin.com/in/chu-kuo-lung)
- **YouTube 頻道**：[雲談風清 - YouTube](https://www.youtube.com/channel/UCXDqLTdCMiCJ1j8xGRfwEig)
- **風清雲談 部落格**：[風清雲談](https://blog.fengqing.tw/)
- **電子郵件**：[fengqing.tw@gmail.com](mailto:fengqing.tw@gmail.com)

---

**📅 最後更新：2025-08-29**  
**👨‍💻 維護者：風清雲談技術團隊**
