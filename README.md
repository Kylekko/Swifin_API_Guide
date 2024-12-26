# Swifin API 개발자 가이드

## 소개

Swifin API는 FastAPI 프레임워크를 기반으로 구축되었으며, 주식 시장 데이터, 경제 지표, 기술 지표 등 다양한 금융 정보를 제공합니다.

**Swifin API 도메인:** `https://swifin.app/v1` 
(모든 API 요청은 이 도메인을 기반으로 합니다.)

## 목차

1.  **API 개요**
2.  **인증**
3.  **API 엔드포인트**
    -   **사용자 (Users)**
        -   내 정보 조회 (`/users/me`)
    -   **주식 (Stocks)**
        -   주식 상세 정보 조회 (`/stocks/{ticker}`)
    -   **주식 행동 (Actions)**
        -   주식 행동 기록 조회 (`/actions/{ticker_symbol}`)
    -   **일별 주가 (Daily Prices)**
        -   일별 주가 정보 조회 (`/daily_prices/{ticker}`)
    -   **재무제표 (Financial Statements)**
        -   재무제표 항목 조회 (`/financial_statements/{ticker_symbol}/{statement_type}/{period}`)
        -   재무제표 정보 조회 (`/financial_statements/{ticker_symbol}/{statement_type}/{period}/{item}`)
    -   **기관 투자자 (Institutional Holders)**
        -   기관 투자자 정보 조회 (`/institutional_holders/{ticker_symbol}`)
    -   **주요 주주 (Major Holders)**
        -   주요 주주 정보 조회 (`/major_holders/{ticker_symbol}`)
    -   **SEC 공시 (SEC Filings)**
        -   SEC 공시 정보 조회 (`/sec_filings/{ticker}`)
    -   **기술 지표 (Technical Indicators)**
        -   기술 지표 정보 조회 (`/technical_indicators/{ticker}/{indicator_type}`)
    -   **티커 (Tickers)**
        -   티커 정보 조회 (`/tickers/{ticker_symbol}/{category}/{subcategory}`)
    -   **경제 지표 (Economic Indicators)**
        -   경제 지표 조회 (`/economic_indicators/`)
    -   **시장 심리 (Market Sentiment)**
        -   시장 심리 조회 (`/market_sentiment/`)
4.  **데이터 모델**
    -   **Users**
    -   **Actions**
    -   **DailyPrices**
    -   **EconomicIndicators**
    -   **FinancialStatements**
    -   **InstitutionalHolders**
    -   **MajorHolders**
    -   **PromoCodes**
    -   **SecFilings**
    -   **Stocks**
    -   **TechnicalIndicators**
    -   **Tickers**
5.  **요청 제한**
6.  **오류 처리**
7.  **샘플 코드**
8.  **참고사항**

----------

## 1. API 개요

-   **주식 데이터**: 주식 가격, 재무제표, 기관 투자자 정보 등
-   **경제 지표**: 주요 경제 지표 데이터
-   **기술 지표**: 주식 기술 분석에 사용되는 지표 데이터
-   **요청 제한**: API 남용 방지를 위한 요청 제한 기능

모든 API 요청은 `https://swifin.app/v1`을 기본 URL로 사용합니다.

## 2. 인증

API 요청 시 Bearer 토큰을 사용하여 인증해야 합니다. 토큰은 Google OAuth 2.0을 통해 발급받을 수 있습니다.

### Google OAuth를 통한 인증 절차

1.  `https://swifin.app/auth/google/login` 엔드포인트로 `GET` 요청을 보내면, Swifin 애플리케이션이 Google 로그인 페이지로 리디렉션합니다.
2.  Google 계정으로 로그인하면, Swifin 애플리케이션의 설정된 `REDIRECT_URI` (예: `https://swifin.app/auth/google/callback`)로 리디렉션됩니다.
4.  백엔드 서버는 이 리디렉션 URL에서 사용자 정보를 처리하고 액세스 토큰(Access Token)을 발급합니다.
5.  액세스 토큰은 `Bearer` 토큰 형식으로 발급되며, 모든 API 요청 시 `Authorization` 헤더에 포함하여 사용합니다.

### 인증 헤더 예시

```
Authorization: <your_access_token>
```

**중요**: `<your_access_token>` 부분을 실제 발급받은 액세스 토큰으로 대체해야 합니다.

## 3. API 엔드포인트

### 사용자 (Users)

### 내 정보 조회

-   **엔드포인트**: `/users/me`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 Header**: <- CURL등
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급 받은 액세스 토큰을 포함합니다.
-   **요청 Body**: 없음 (요청 시 별도의 데이터 전송 필요 없음)
    
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        {
          "email": "user@swifin.app",
          "subscription_plan": "free",
          "subscription_start": "2024-07-18T00:00:00",
          "subscription_end": "2024-08-17T00:00:00",
          "access_token": "your_access_token",
          "request_limit": 100,
          "current_requests": 0
        }
        ```
        
    -   **실패 (401 Unauthorized)**:
        
        ```json
        {
          "detail": "자격 증명을 확인할 수 없습니다."
        }
        ```
        
-   **설명**: 현재 인증된 사용자의 상세 정보를 조회합니다.
    
    -   `email`: 사용자 이메일 주소
    -   `subscription_plan`: 구독 플랜 (free, basic, pro)
    -   `subscription_start`: 구독 시작 날짜 및 시간
    -   `subscription_end`: 구독 종료 날짜 및 시간
    -   `access_token`: 현재 사용 중인 액세스 토큰
    -   `request_limit`: API 요청 제한 횟수
    -   `current_requests`: 현재 API 요청 횟수

### 인증 (Auth)

### Google 로그인

-   **엔드포인트**: `/auth/google/login`
-   **HTTP 메소드**: `GET`
-   **요청 헤더**: 없음
-   **요청 바디**: 없음
-   **응답**:
    -   Google 로그인 페이지로 리디렉션 (브라우저에서 자동으로 이루어짐)
-   **설명**: 사용자를 Google 로그인 페이지로 리디렉션하여 인증 절차를 시작합니다.

### Google 콜백

-   **엔드포인트**: `/auth/google/callback`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**: 없음
    
-   **요청 바디**: 없음 (Google에서 자동으로 전달하는 인증 정보가 포함)
    
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        {
          "access_token": "your_access_token",
          "token_type": "bearer"
        }
        ```
        
    -   **실패 (400 Bad Request)**:
        
        ```json
        {
            "detail": "Google 인증 실패"
        }
        ```
        
-   **설명**: Google OAuth 인증 후 액세스 토큰을 발급합니다. 이 토큰은 이후 API 요청 시 `Authorization` 헤더에 사용됩니다.
    
    -   `access_token`: 발급된 액세스 토큰
    -   `token_type`: 토큰 타입 (항상 "bearer")

### 주식 (Stocks)

### 주식 상세 정보 조회

-   **엔드포인트**: `/stocks/{ticker}`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**:
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
    
-   **경로 파라미터**:
    
    -   `ticker` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        {
          "ticker": "AAPL",
          "name": "Apple Inc.",
          "sector": "Technology",
          "industry": "Consumer Electronics",
           "exchange": "NASDAQ",
           "pb_ratio": 15.0000,
           "pe_ratio": 25.0000
        }
        ```
        
    -   **실패 (401 Unauthorized)**:
        
        ```json
        {
            "detail": "자격 증명을 확인할 수 없습니다."
        }
        ```
        
    -   **실패 (404 Not Found)**:
        
        ```json
        {
          "detail": "주식을 찾을 수 없습니다."
        }
        ```
        
-   **설명**: 특정 티커의 주식 상세 정보를 조회합니다.
    
    -   `ticker`: 주식 티커 심볼
    -   `name`: 회사 이름
    -   `sector`: 주식 섹터 (예: Technology, Healthcare 등)
    -   `industry`: 주식 산업 (예: Consumer Electronics, Biotechnology 등)
    -   `exchange`: 주식 거래소 (예: NASDAQ, NYSE 등)
    -   `pb_ratio`: 주가순자산비율 (P/B ratio)
    -   `pe_ratio`: 주가수익비율 (P/E ratio)

### 주식 목록 조회

-   **엔드포인트**: `/stocks/`
-   **HTTP 메소드**: `GET`
-   **요청 헤더**:
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
-   **쿼리 파라미터**:
    -   `skip` (int, 선택): 건너뛸 데이터 수입니다. 페이지네이션을 위해 사용합니다. 기본값은 `0`입니다. 예: `skip=10`
    -   `limit` (int, 선택): 가져올 데이터 수입니다. 페이지네이션을 위해 사용합니다. 기본값은 `100`입니다. 예: `limit=50`
-   **응답**:
```Json
[ 
	{ 
		"ticker": "AAPL", 
		"name": "Apple Inc.", 
		"sector": "Technology", 
		"industry": "Consumer Electronics", 
		"exchange": "NASDAQ", 
		"pb_ratio": 15.0000, 
		"pe_ratio": 25.0000 
	}
]
```
        
- **실패 (401 Unauthorized)**:
        
```Json
{
	"detail": "자격 증명을 확인할 수 없습니다."
}
```
        
- **실패 (404 Not Found)**:
        
```Json
{
	"detail": "데이터가 존재하지 않습니다."
}
```
        
- **설명**: 주식 목록을 페이지네이션하여 조회합니다. 응답에는 각 주식의 티커, 회사명, 섹터, 산업, 거래소, P/B 비율, P/E 비율이 포함됩니다.

### 주식 행동 (Actions)

### 주식 행동 기록 조회

- **엔드포인트**: `/actions/{ticker_symbol}`
- **HTTP 메소드**: `GET`
- **요청 헤더**:
    - `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
- **요청 바디**: 없음
- **경로 파라미터**:
    - `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
- **쿼리 파라미터**:
    - `period` (string, 선택): 조회할 기간을 설정합니다. 다음 값 중 하나를 입력할 수 있습니다:
        - `1y`: 1년 간의 데이터
        - `3y`: 3년 간의 데이터
        - `5y`: 5년 간의 데이터
        - `all`: 전체 기간의 데이터 
          (기본값입니다. `period` 파라미터를 생략하면 `all`과 동일하게 동작합니다)
- **응답**:
    - **성공 (200 OK)**:
        
```json
[
    {
        "ticker_symbol": "AAPL",
        "date": "2023-01-05",
        "dividends": 0.2,
        "stock_splits": null
    },
     {
        "ticker_symbol": "AAPL",
        "date": "2022-06-05",
        "dividends": 0.3,
         "stock_splits": 2
    }
]
```
        
    - **실패 (401 Unauthorized)**:
        
```json
{
   "detail": "자격 증명을 확인할 수 없습니다."
}
```
        
    - **실패 (404 Not Found)**:
        
```json
{
   "detail": "데이터를 찾을 수 없습니다."
}
```
        
- **설명**: 특정 티커의 주식 행동 기록 (배당금, 주식 분할 등)을 조회합니다.
    - `ticker_symbol`: 주식 티커 심볼
    - `date`: 해당 행동이 발생한 날짜
    - `dividends`: 배당금
    - `stock_splits`: 주식 분할 비율

### 일별 주가 (Daily Prices)

### 일별 주가 정보 조회

- **엔드포인트**: `/daily_prices/{ticker}`
- **HTTP 메소드**: `GET`
- **요청 헤더**:
    - `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
- **요청 바디**: 없음
- **경로 파라미터**:
    - `ticker` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
- **쿼리 파라미터**:
    - `period` (string, 필수): 조회할 기간을 설정합니다. 다음 값 중 하나를 입력해야 합니다:
        - `5d`: 최근 5일 데이터
        - `1m`: 최근 1개월 데이터
        - `3m`: 최근 3개월 데이터
        - `6m`: 최근 6개월 데이터
        - `1y`: 최근 1년 데이터
- **응답**:
```json
[
	{
		"ticker": "AAPL",
		"date": "2024-07-18",
		"open": 150.0000,
		"high": 152.0000,
		"low": 149.0000,
		"close": 151.0000,
		"volume": 100000
	},
	{
		"ticker": "AAPL",
		"date": "2024-07-17",
		"open": 148.0000,
		"high": 150.0000,
		"low": 147.0000,
		"close": 149.0000,
		"volume": 110000
	}
]
```
  
- **실패 (401 Unauthorized)**:
   
``` json
{
   "detail": "자격 증명을 확인할 수 없습니다."
}  
```
    
- **실패 (404 Not Found)**:
    
 ```json
{
   "detail": "데이터를 찾을 수 없습니다."
}
 ```

-   **설명**: 특정 티커의 일별 주가 정보 (시가, 고가, 저가, 종가, 거래량)를 조회합니다.
    -   `ticker`: 주식 티커 심볼
    -   `date`: 해당 날짜
    -   `open`: 시가
    -   `high`: 고가
    -   `low`: 저가
    -   `close`: 종가
    -   `volume`: 거래량

### 재무제표 (Financial Statements)

### 재무제표 항목 조회

-   **엔드포인트**: `/financial_statements/{ticker_symbol}/{statement_type}/{period}`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**:
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
    
-   **경로 파라미터**:
    
    -   `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
    -   `statement_type` (string, 필수): 재무제표 유형입니다. 다음 값 중 하나를 입력해야 합니다:
        -   `balance_sheet`: 대차대조표
        -   `financials`: 손익계산서
        -   `cashflow`: 현금흐름표
    -   `period` (string, 필수): 회계 기간을 나타냅니다. 다음 값 중 하나를 입력해야 합니다:
        -   `annual`: 연간
        -   `quarterly`: 분기별
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        [
          "Accounts Payable",
          "Accounts Receivable",
          "Accumulated Depreciation",
          "Available For Sale Securities",
          "Capital Lease Obligations",
          "Capital Stock",
          "Cash And Cash Equivalents",
          "Cash Cash Equivalents And Short Term Investments",
          "Cash Equivalents",
          "Cash Financial",
          "Commercial Paper",
          "Common Stock",
          "Common Stock Equity",
          "Current Assets",
          "Current Capital Lease Obligation",
           "Current Debt",
          "Current Debt And Capital Lease Obligation",
          "Current Deferred Liabilities",
          "Current Deferred Revenue",
          "Current Liabilities",
          "Gains Losses Not Affecting Retained Earnings",
          "Gross PPE",
          "Income Tax Payable",
          "Inventory",
          "Invested Capital",
          "Investmentin Financial Assets",
          "Investments And Advances",
          "Land And Improvements",
          "Leases",
          "Long Term Capital Lease Obligation",
          "Long Term Debt",
          "Long Term Debt And Capital Lease Obligation",
          "Machinery Furniture Equipment",
          "Net Debt",
          "Net PPE",
          "Net Tangible Assets",
          "Non Current Deferred Assets",
          "Non Current Deferred Taxes Assets",
          "Ordinary Shares Number",
          "Other Current Assets",
          "Other Current Borrowings",
          "Other Current Liabilities",
          "Other Equity Adjustments",
          "Other Investments",
          "Other Non Current Assets",
          "Other Non Current Liabilities",
          "Other Properties",
          "Other Receivables",
          "Other Short Term Investments",
          "Payables",
          "Payables And Accrued Expenses",
          "Properties",
          "Receivables",
          "Retained Earnings",
          "Share Issued",
          "Stockholders Equity",
          "Tangible Book Value",
          "Total Assets",
          "Total Capitalization",
          "Total Debt",
          "Total Equity Gross Minority Interest",
          "Total Liabilities Net Minority Interest",
          "Total Non Current Assets",
           "Total Non Current Liabilities Net Minority Interest",
          "Total Tax Payable",
          "Tradeand Other Payables Non Current",
          "Treasury Shares Number",
          "Working Capital"
        ]
        
        
        ```
        
    -   **실패 (401 Unauthorized)**:
        
        ```json
        {
            "detail": "자격 증명을 확인할 수 없습니다."
        }
        ```

        
    -   **실패 (404 Not Found)**:
        
        ```json
        {
            "detail": "데이터를 찾을 수 없습니다."
        }
        ```
        
-   **설명**: 특정 티커, 재무제표 유형 및 회계 기간에 해당하는 재무제표 항목 목록을 조회합니다. 재무제표 데이터를 특정 항목별로 조회하기 전에 사용 가능한 항목을 확인하는 데 유용합니다.
    

### 재무제표 정보 조회

-   **엔드포인트**: `/financial_statements/{ticker_symbol}/{statement_type}/{period}/{item}`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**:
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
    
-   **경로 파라미터**:
    
    -   `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
    -   `statement_type` (string, 필수): 재무제표 유형입니다. 다음 값 중 하나를 입력해야 합니다:
        -   `balance_sheet`: 대차대조표
        -   `financials`: 손익계산서
        -   `cashflow`: 현금흐름표
    -   `period` (string, 필수): 회계 기간을 나타냅니다. 다음 값 중 하나를 입력해야 합니다:
        -   `annual`: 연간
        -   `quarterly`: 분기별
    -   `item` (string, 필수): 조회하려는 재무제표 항목입니다. 예: `totalAssets`, `revenue`, `netCashFlow`. 사용가능한 item은 `/financial_statements/{ticker_symbol}/{statement_type}/{period}`엔드포인트를 이용하여 확인하세요.
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        [
             {
                "ticker_symbol": "AAPL",
                "statement_type": "balance_sheet",
                "period": "annual",
                "date": "2023-12-31",
                "item": "totalAssets",
                "value": 350000000000
            },
             {
                "ticker_symbol": "AAPL",
                "statement_type": "balance_sheet",
                 "period": "annual",
                "date": "2022-12-31",
                "item": "totalAssets",
                "value": 320000000000
             }
        ]
        ```
        
    -   **실패 (401 Unauthorized)**:
        
        ```json
        {
            "detail": "자격 증명을 확인할 수 없습니다."
        }
        ```
        
    -   **실패 (404 Not Found)**:
        
        ```json
        {
          "detail": "데이터를 찾을 수 없습니다."
        }
        ```
        
-   **설명**: 특정 티커의 특정 재무제표 항목 데이터를 조회합니다.
    
    -   `ticker_symbol`: 주식 티커 심볼
    -   `statement_type`: 재무제표 유형
    -   `period`: 회계 기간
    -   `date`: 재무제표 데이터 날짜
    -   `item`: 재무제표 항목
    -   `value`: 해당 항목의 값

### 기관 투자자 (Institutional Holders)

### 기관 투자자 정보 조회

-   **엔드포인트**: `/institutional_holders/{ticker_symbol}`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**:
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
    
-   **경로 파라미터**:
    
    -   `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        [
          {
            "ticker_symbol": "AAPL",
            "date_reported": "2024-03-31",
            "holder": "Vanguard Group Inc",
            "pct_held": 8.0000,
            "shares": 130000000,
            "value": 15000000000
         },
          {
            "ticker_symbol": "AAPL",
            "date_reported": "2024-03-31",
            "holder": "BlackRock Inc.",
            "pct_held": 7.0000,
            "shares": 120000000,
            "value": 14000000000
          }
        ]
        ```
        
    -   **실패 (401 Unauthorized)**:
        
        ```json
        {
            "detail": "자격 증명을 확인할 수 없습니다."
        }
        ```
        
    -   **실패 (404 Not Found)**:
        
        ```json
        {
           "detail": "데이터를 찾을 수 없습니다."
        }
        ```
        
-   **설명**: 특정 티커의 기관 투자자 정보를 조회합니다.
    
    -   `ticker_symbol`: 주식 티커 심볼
    -   `date_reported`: 보고 날짜
    -   `holder`: 기관 투자자 이름
    -   `pct_held`: 보유 비율 (%)
    -   `shares`: 보유 주식 수
    -   `value`: 보유 가치

### 주요 주주 (Major Holders)

### 주요 주주 정보 조회

-   **엔드포인트**: `/major_holders/{ticker_symbol}`
-   **HTTP 메소드**: `GET`
-   **요청 헤더**:
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
-   **경로 파라미터**:
    -   `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
-   **응답**:
```json
[ 
	{ 
		"ticker_symbol": "AAPL", 
		"breakdown": "Top Institutional Holders", 
		"value": 65.3 
	}
]
```        
- **실패 (401 Unauthorized)**:
        
```json
{
    "detail": "자격 증명을 확인할 수 없습니다."
}
```
-   **실패 (404 Not Found)**:
```json
{
   "detail": "데이터를 찾을 수 없습니다."
}
```
    
- **설명**: 특정 티커의 주요 주주 정보를 조회합니다.
    - `ticker_symbol`: 주식 티커 심볼
    - `breakdown`: 주주 유형 (예: Top Institutional Holders, Top Mutual Fund Holders 등)
    - `value`: 해당 주주 유형이 보유한 주식 가치 (%)

### SEC 공시 (SEC Filings)

### SEC 공시 정보 조회

- **엔드포인트**: `/sec_filings/{ticker}`
- **HTTP 메소드**: `GET`
- **요청 헤더**:
    - `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
- **요청 바디**: 없음
- **경로 파라미터**:
    - `ticker` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
- **쿼리 파라미터**:
    - `form_type` (string, 선택): 조회할 공시 유형을 설정합니다. 다음 값 중 하나를 입력할 수 있습니다:
        - `4`: 내부자 거래 보고서
        - `13`: 주요 주주 보고서
        - `8-K`: 중요한 사건 보고서
        - `10-Q`: 분기별 보고서
        - `10-K`: 연간 보고서
        - `form_type` 파라미터를 생략하면 해당 티커의 모든 공시 유형을 조회합니다.
- **응답**:
    - **성공 (200 OK)**:
```json
[
    {
        "ticker": "AAPL",
        "cik": "0000320193",
        "accession_number": "000119312523196117",
        "form_type": "4",
        "filing_date": "2023-07-18",
        "important_content": "some text content here..."
     },
      {
        "ticker": "AAPL",
        "cik": "0000320193",
        "accession_number": "000119312523196117",
        "form_type": "10-Q",
        "filing_date": "2023-06-30",
        "important_content": "some text content here..."
      }
]
```
- **실패 (401 Unauthorized)**:
        
```json
{
    "detail": "자격 증명을 확인할 수 없습니다."
}
```
        
- **실패 (404 Not Found)**:
        
 ```json
{
   "detail": "데이터를 찾을 수 없습니다."
}
 ```
- **설명**: 특정 티커의 SEC 공시 정보를 조회합니다.
    - `ticker`: 주식 티커 심볼
    - `cik`: SEC 기업 식별 코드 (CIK)
    - `accession_number`: SEC 문서 접근 번호
    - `form_type`: 공시 유형 (예: 4, 13, 8-K, 10-Q, 10-K)
    - `filing_date`: 공시 날짜
    - `important_content`: 공시의 중요 내용 발췌

### 기술 지표 (Technical Indicators)

### 기술 지표 정보 조회

- **엔드포인트**: `/technical_indicators/{ticker}/{indicator_type}`
- **HTTP 메소드**: `GET`
- **요청 헤더**:
    - `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
- **요청 바디**: 없음
- **경로 파라미터**:
    - `ticker` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
    - `indicator_type` (string, 필수): 조회하려는 기술 지표 유형입니다. 예: `SMA_50, SMA_200, EMA_12, EMA_26, RSI, PB_RATIO, PE_RATIO` (단순 이동 평균).
- **쿼리 파라미터**:
    - `period` (string, 필수): 조회할 기간을 설정합니다. 다음 값 중 하나를 입력해야 합니다:
        - `1d`: 1일 데이터
        - `1w`: 1주 데이터
        - `1m`: 1개월 데이터
        - `3m`: 3개월 데이터
        - `6m`: 6개월 데이터
        - `1y`: 1년 데이터
- **응답**:
    - **성공 (200 OK)**:
        
```json
[
 {
        "ticker": "AAPL",
        "date": "2024-07-18",
        "indicator_type": "SMA",
        "value": 150.0000
  },
   {
        "ticker": "AAPL",
        "date": "2024-07-17",
         "indicator_type": "SMA",
        "value": 149.0000
   }
]
```
- **실패 (401 Unauthorized)**:
 
```json
{
    "detail": "자격 증명을 확인할 수 없습니다."
}
```
 
- **실패 (404 Not Found)**:
 
```json
{
    "detail": "데이터를 찾을 수 없습니다."
}
```

### 티커 (Tickers)

#### 티커 정보 조회

*   **엔드포인트**: `/tickers/{ticker_symbol}/{category}/{subcategory}`
*   **HTTP 메소드**: `GET`
*   **요청 헤더**:
    *   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
*   **요청 바디**: 없음
*   **경로 파라미터**:
    *   `ticker_symbol` (string, 필수): 조회하려는 주식의 티커 심볼입니다. 예: `AAPL`, `GOOG`, `TSLA`. 대문자로 입력해야 합니다.
    *   `category` (string, 필수): 조회하려는 데이터의 큰 분류입니다. 다음 값 중 하나를 입력해야 합니다:
        *   `company_info`: 회사 기본 정보
        *   `stock_price`: 주식 가격 관련 정보
        *   `financials`: 재무 관련 정보
        *   `dividends`: 배당 관련 정보
        *   `valuation`: 밸류에이션 관련 정보
        *   `market_sentiment`: 시장 심리 관련 정보
        *   `management`: 경영진 관련 정보
    *    `subcategory` (string, 필수): `category`에 따라 세분화된 하위 분류입니다. 각 `category`에 따른 가능한 값은 아래 표를 참고하세요.
*   **응답**:
    *   **성공 (200 OK)**:
        ```json
        {
          "ticker_symbol": "AAPL",
          "data": {
            "name": "Apple Inc.",
            "sector": "Technology",
            "industry": "Consumer Electronics"
          }
        }
        ```
    *   **실패 (400 Bad Request)**:
        ```json
        {
          "detail": "잘못된 카테고리입니다. 가능한 카테고리: ['company_info', 'stock_price', 'financials', 'dividends', 'valuation', 'market_sentiment', 'management']"
        }
        ```
        ```json
        {
          "detail": "잘못된 서브카테고리입니다. 가능한 서브카테고리: ['identification', 'industry']"
        }
        ```
    *  **실패 (401 Unauthorized)**:
       ```json
        {
            "detail": "자격 증명을 확인할 수 없습니다."
        }
       ```
    *   **실패 (404 Not Found)**:
        ```json
        {
          "detail": "티커를 찾을 수 없습니다."
        }
        ```
*   **설명**: 특정 티커의 상세 정보를 카테고리 및 서브카테고리에 따라 조회합니다.

**가능한 카테고리 및 서브카테고리:**

| category         | subcategory        | 설명                                                                                               | 포함 필드                                                                                                                                                                                                      |
| :--------------- | :----------------- | :------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `company_info`   | `identification`  | 회사 기본 정보 (티커, 회사명, 사업 개요)                                                                   | `ticker_symbol`, `name`, `longBusinessSummary`                                                                                                                                                                   |
|                  | `industry`        | 회사 산업 정보 (섹터, 산업, 거래소)                                                                    | `sector`, `industry`, `exchange`                                                                                                                                                                               |
| `stock_price`    | `price_movement`  | 현재 가격 및 일중 변동 정보                                                                         | `currentPrice`, `open`, `dayHigh`, `dayLow`, `previousClose`, `volume`                                                                                                                                           |
|                  | `price_trend`     | 가격 추세 관련 정보                                                                               | `fiftyTwoWeekHigh`, `fiftyTwoWeekLow`, `52WeekChange`, `averageVolume`, `averageVolume10days`, `beta`                                                                                                              |
| `financials`     | `profitability`  | 수익성 관련 정보                                                                                   | `totalRevenue`, `revenuePerShare`, `revenueGrowth`, `grossMargins`, `ebitda`, `ebitdaMargins`, `operatingMargins`, `profitMargins`, `netIncomeToCommon`, `earningsGrowth`, `earningsQuarterlyGrowth`  |
|                  | `per_share`       | 주당 관련 정보                                                                                      | `trailingEps`, `forwardEps`, `totalCashPerShare`, `bookValue`                                                                                                                                                 |
|                  | `valuation_metrics`  | 밸류에이션 지표 관련 정보                                                                                 | `trailingPE`, `forwardPE`, `trailingPegRatio`, `priceToSalesTrailing12Months`, `priceToBook`, `enterpriseToEbitda`, `enterpriseToRevenue`                                                                    |
|                  | `financial_health` | 재정 건전성 관련 정보                                                                                   | `totalDebt`, `totalCash`, `debtToEquity`, `currentRatio`, `quickRatio`                                                                                                                                    |
|                  | `cash_flow`       | 현금 흐름 관련 정보                                                                                    | `freeCashflow`, `operatingCashflow`                                                                                                                                                                        |
|                  | `efficiency`      | 효율성 관련 정보                                                                                      | `returnOnAssets`, `returnOnEquity`                                                                                                                                                                         |
| `dividends`      | `dividend_policy` | 배당 정책 관련 정보                                                                                  | `dividendRate`, `dividendYield`, `trailingAnnualDividendRate`, `trailingAnnualDividendYield`, `payoutRatio`, `fiveYearAvgDividendYield`, `lastDividendDate`, `exDividendDate`                            |
|                  | `shares`          | 발행 주식 수 관련 정보                                                                                 | `sharesOutstanding`, `floatShares`                                                                                                                                                                      |
| `valuation`      | `price_targets`   | 목표 주가 관련 정보                                                                                    | `targetMeanPrice`, `targetHighPrice`, `targetLowPrice`, `targetMedianPrice`                                                                                                                                    |
|                  | `recommendations` | 분석가 추천 정보                                                                                     | `numberOfAnalystOpinions`, `recommendationMean`, `recommendationKey`                                                                                                                                         |
| `market_sentiment`| `short_interest` | 공매도 관련 정보                                                                                     | `shortRatio`, `sharesShort`, `shortPercentOfFloat`, `sharesShortPriorMonth`, `dateShortInterest`                                                                                                        |
|                  | `enterprise_value`| 기업 가치 관련 정보                                                                                    | `marketCap`, `enterpriseValue`                                                                                                                                                                          |
|                  | `ownership`       | 소유권 관련 정보                                                                                      | `heldPercentInstitutions`, `heldPercentInsiders`                                                                                                                                                             |
| `management`     | `executives`      | 경영진 정보                                                                                       | `companyOfficers`                                                                                                                                                                                          |
|                  | `risk_factors`   | 리스크 요인 관련 정보                                                                                   | `auditRisk`, `boardRisk`, `compensationRisk`, `shareHolderRightsRisk`, `overallRisk`                                                                                                                        |
|                  | `additional_info` | 추가 정보                                                                                         | `fullTimeEmployees`                                                                                                                                                                                           |

### 경제 지표 (Economic Indicators)

#### 경제 지표 조회

*   **엔드포인트**: `/economic_indicators/`
*   **HTTP 메소드**: `GET`
*   **요청 헤더**:
    *   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
*   **요청 바디**: 없음
*   **쿼리 파라미터**:
    *   `limit` (int, 선택): 조회할 데이터 수입니다. 기본값은 `100`입니다. 예: `limit=200`
*   **응답**:
    *   **성공 (200 OK)**:
        ```json
        [
         {
            "id": 1,
            "indicator_name": "GDP",
            "date": "2024-06-30",
            "value": 25000,
            "frequency": "quarterly"
         },
        {
            "id": 2,
            "indicator_name": "CPI",
             "date": "2024-06-30",
            "value": 3.5,
            "frequency": "monthly"
          }
        ]
        ```
     *   **실패 (401 Unauthorized)**:
	        ```json
	        {
	           "detail": "자격 증명을 확인할 수 없습니다."
	        }
	        ```
    *   **실패 (404 Not Found)**:
        ```json
        {
          "detail": "데이터를 찾을 수 없습니다."
        }
        ```
*   **설명**: 경제 지표 데이터를 조회합니다.
    *   `id`: 경제 지표 고유 ID
    *   `indicator_name`: 경제 지표 이름 (예: GDP, CPI)
    *   `date`: 발표 날짜
    *   `value`: 해당 경제 지표 값
    *   `frequency`: 발표 빈도 (예: monthly, quarterly)
### 시장 심리 (Market Sentiment)

### 시장 심리 조회

-   **엔드포인트**: `/market_sentiment/`
    
-   **HTTP 메소드**: `GET`
    
-   **요청 헤더**:
    
    -   `Authorization: Bearer <your_access_token>` (필수): 발급받은 액세스 토큰을 포함합니다.
-   **요청 바디**: 없음
    
-   **쿼리 파라미터**:
    
    -   `limit` (int, 선택): 조회할 데이터 수입니다. 기본값은 `100`입니다. 예: `limit=200`
-   **응답**:
    
    -   **성공 (200 OK)**:
        
        ```json
        [
           {
              "id": 1,
              "date": "2024-07-18",
              "vix": 15.5,
              "fear_greed_index": 60.0
           },
          {
            "id": 2,
            "date": "2024-07-17",
            "vix": 16.0,
            "fear_greed_index": 55.0
           }
        ]
        ```
        
    -   **실패 (401 Unauthorized)**:
        
    
	    ```json
	     {
	       "detail": "자격 증명을 확인할 수 없습니다."
	     }
	    ```
    
    -   **실패 (404 Not Found)**:
        
        ```json
        {
           "detail": "데이터를 찾을 수 없습니다."
        }
        
        
        ```
        
-   **설명**: 시장 심리 데이터를 조회합니다.
    
    -   `id`: 시장 심리 데이터 고유 ID
    -   `date`: 데이터 날짜
    -   `vix`: 변동성 지수 (VIX) 값
    -   `fear_greed_index`: 공포-탐욕 지수 값

## 4. 요청 제한

Swifin API는 API 남용을 방지하기 위해 요청 제한을 적용하고 있습니다. 각 사용자 플랜에 따라 API 요청 횟수가 제한되며, 요청 횟수가 초과될 경우 `429 Too Many Requests` 에러가 발생합니다.

-   **Free 플랜**: 1일 100회 요청 제한
	-  stocks
	- actions
	- daily_price
	- financial_statements
	- institutional_holders
	- major_holders
	- technical_indicators
	- economic_indicators
	- market_sentiment
-   **Basic 플랜**: 1일 1000회 요청 제한
	-  stocks
	- actions
	- daily_price
	- financial_statements
	- institutional_holders
	- major_holders
	- technical_indicators
	- economic_indicators
	- market_sentiment
-   **Pro 플랜**: 1일 5000회 요청 제한
	-  stocks
	- actions
	- daily_price
	- financial_statements
	- institutional_holders
	- major_holders
	- technical_indicators
	- economic_indicators
	- market_sentiment
	- (추가) sec_filings
		- Summary import contents
	- (추가) tickers

요청 제한은 사용자 이메일과 토큰, 결제내역을 기준으로 관리되며, 매일 자정에 초기화됩니다.

## 5. 오류 처리

API 요청 실패 시, 다음과 같은 HTTP 상태 코드 및 오류 메시지가 반환될 수 있습니다.

-   `400 Bad Request`: 잘못된 요청 (예: 유효하지 않은 파라미터, 필수 파라미터 누락)
-   `401 Unauthorized`: 인증 실패 (예: 액세스 토큰 누락, 유효하지 않은 토큰)
-   `403 Forbidden`: 권한 없음 (예: 비활성화된 계정, 플랜 미구독)
-   `404 Not Found`: 데이터 찾을 수 없음
-   `429 Too Many Requests`: 요청 제한 초과
-   `500 Internal Server Error`: 서버 내부 오류

오류 응답은 다음과 같은 JSON 형식으로 반환됩니다:

```json
{
  "detail": "오류 메시지"
}
```

## 6. 샘플 코드

추후 다양한 언어(Python, JavaScript 등)를 사용한 샘플 코드를 제공할 예정입니다.

## Financial Statements 카테고리의 모든 Item - 개별 호출

아래는 `financial_statements` 엔드포인트에서 사용가능한 `item` 값들입니다. 이 값들은 각 `statement_type` (balance_sheet, financials, cashflow) 와 `period` (annual, quarterly) 에 따라 사용가능할 수도, 사용 불가능할 수도 있습니다.

* **balance_sheet**
    *  Accounts Payable
    *  Accounts Receivable
    *  Accumulated Depreciation
    *  Available For Sale Securities
    *  Capital Lease Obligations
    *  Capital Stock
    *  Cash And Cash Equivalents
    *  Cash Cash Equivalents And Short Term Investments
    *  Cash Equivalents
    *  Cash Financial
    *  Commercial Paper
    *  Common Stock
    *  Common Stock Equity
    *  Current Assets
    *  Current Capital Lease Obligation
    *  Current Debt
    *  Current Debt And Capital Lease Obligation
    *  Current Deferred Liabilities
    *  Current Deferred Revenue
    *  Current Liabilities
    *  Gains Losses Not Affecting Retained Earnings
    * Gross PPE
    *  Income Tax Payable
    *  Inventory
    * Invested Capital
    *  Investmentin Financial Assets
    *  Investments And Advances
    * Land And Improvements
    * Leases
    *  Long Term Capital Lease Obligation
    *  Long Term Debt
    *  Long Term Debt And Capital Lease Obligation
    * Machinery Furniture Equipment
    *  Net Debt
    *  Net PPE
    *  Net Tangible Assets
    *  Non Current Deferred Assets
    *  Non Current Deferred Taxes Assets
    * Ordinary Shares Number
    *  Other Current Assets
    * Other Current Borrowings
    *  Other Current Liabilities
    *  Other Equity Adjustments
    *  Other Investments
    *  Other Non Current Assets
    *  Other Non Current Liabilities
    *  Other Properties
    *  Other Receivables
    *  Other Short Term Investments
    * Payables
    *  Payables And Accrued Expenses
    * Properties
    * Raw Materials
    * Receivables
    *  Retained Earnings
    * Share Issued
    *  Stockholders Equity
    *  Tangible Book Value
    *  Total Assets
    *  Total Capitalization
    *  Total Debt
    *  Total Equity Gross Minority Interest
    * Total Liabilities Net Minority Interest
    *  Total Non Current Assets
    * Total Non Current Liabilities Net Minority Interest
    * Total Tax Payable
    * Tradeand Other Payables Non Current
    * Treasury Shares Number
    * Working Capital
 * **financials**
    * Basic Average Shares
    * Basic EPS
    * Cost Of Revenue
    * Diluted Average Shares
    * Diluted EPS
    * Diluted NI Availto Com Stockholders
    * EBIT
    * EBITDA
    * Gross Profit
    * Interest Expense
    * Interest Expense Non Operating
    * Interest Income
    * Interest Income Non Operating
    * Net Income
    * Net Income Common Stockholders
    * Net Income Continuous Operations
    * Net Income From Continuing And Discontinued Operation
    * Net Income From Continuing Operation Net Minority Interest
    * Net Income Including Noncontrolling Interests
    * Net Interest Income
    * Net Non Operating Interest Income Expense
    * Normalized EBITDA
    * Normalized Income
    * Operating Expense
    * Operating Income
    * Operating Revenue
    * Other Income Expense
    * Other Non Operating Income Expenses
    * Pretax Income
    * Reconciled Cost Of Revenue
    * Reconciled Depreciation
    * Research And Development
    * Selling General And Administration
    * Tax Effect Of Unusual Items
    * Tax Provision
    * Tax Rate For Calcs
    * Total Expenses
    * Total Operating Income As Reported
    * Total Revenue

* **cashflow**
    * Beginning Cash Position
    * Capital Expenditure
    * Cash Dividends Paid
    * Cash Flow From Continuing Financing Activities
    * Cash Flow From Continuing Investing Activities
    * Cash Flow From Continuing Operating Activities
    * Change In Account Payable
    * Change In Inventory
    * Change In Other Current Assets
    * Change In Other Current Liabilities
    * Change In Other Working Capital
    * Change In Payable
    * Change In Payables And Accrued Expense
    * Change In Receivables
    * Change In Working Capital
    * Changes In Account Receivables
    * Changes In Cash
    * Common Stock Dividend Paid
    * Common Stock Issuance
    * Common Stock Payments
    * Deferred Income Tax
    * Deferred Tax
    * Depreciation Amortization Depletion
    * Depreciation And Amortization
    * End Cash Position
    * Financing Cash Flow
    * Free Cash Flow
    * Income Tax Paid Supplemental Data
    * Interest Paid Supplemental Data
    * Investing Cash Flow
    * Issuance Of Capital Stock
    * Issuance Of Debt
    * Long Term Debt Issuance
    * Long Term Debt Payments
    * Net Business Purchase And Sale
    * Net Common Stock Issuance
    * Net Income From Continuing Operations
    * Net Investment Purchase And Sale
    * Net Issuance Payments Of Debt
    * Net Long Term Debt Issuance
    * Net Other Financing Charges
    * Net Other Investing Changes
    * Net PPE Purchase And Sale
    * Net Short Term Debt Issuance
    * Operating Cash Flow
    * Other Non Cash Items
    * Purchase Of Business
    * Purchase Of Investment
    * Purchase Of PPE
    * Repayment Of Debt
    * Repurchase Of Capital Stock
    * Sale Of Investment
    * Short Term Debt Payments
    * Stock Based Compensation

## Tickers 카테고리의 모든 subcategory와 해당 필드

아래는 `/tickers/{ticker_symbol}/{category}/{subcategory}` 엔드포인트에서 사용가능한 `category`와 그에 따른 `subcategory` 및 각 `subcategory`에 포함된 필드들입니다.

### Company Info
* **identification**
    * `ticker_symbol`
    * `name`
    * `longBusinessSummary`
* **industry**
    * `sector`
    * `industry`
    * `exchange`

### stock_price
* **price_movement**
    * `currentPrice`
    * `open`
    * `dayHigh`
    * `dayLow`
    * `previousClose`
    * `volume`
*   **price_trend**
    * `fiftyTwoWeekHigh`
    * `fiftyTwoWeekLow`
    * `52WeekChange`
    * `averageVolume`
    * `averageVolume10days`
    * `beta`

### financials
*   **profitability**
    *   `totalRevenue`
    *   `revenuePerShare`
    *   `revenueGrowth`
    *   `grossMargins`
    *   `ebitda`
    *   `ebitdaMargins`
    *   `operatingMargins`
    *   `profitMargins`
    *   `netIncomeToCommon`
    *   `earningsGrowth`
    *    `earningsQuarterlyGrowth`
*  **per_share**
    *   `trailingEps`
    *   `forwardEps`
    *   `totalCashPerShare`
    *   `bookValue`
*   **valuation_metrics**
    *   `trailingPE`
    *   `forwardPE`
    *   `trailingPegRatio`
    *   `priceToSalesTrailing12Months`
    *   `priceToBook`
    *   `enterpriseToEbitda`
    *   `enterpriseToRevenue`
*  **financial_health**
    *   `totalDebt`
    *   `totalCash`
    *   `debtToEquity`
    *   `currentRatio`
    *   `quickRatio`
*   **cash_flow**
     *   `freeCashflow`
    *  `operatingCashflow`
*  **efficiency**
    * `returnOnAssets`
    * `returnOnEquity`

### dividends

*   **dividend_policy**
    *   `dividendRate`
    *   `dividendYield`
    *   `trailingAnnualDividendRate`
    *   `trailingAnnualDividendYield`
    *    `payoutRatio`
    * `fiveYearAvgDividendYield`
    * `lastDividendDate`
     *  `exDividendDate`

*   **shares**
    *   `sharesOutstanding`
    *   `floatShares`

### valuation

*   **price_targets**
    *   `targetMeanPrice`
    *   `targetHighPrice`
    *   `targetLowPrice`
    *  `targetMedianPrice`
*   **recommendations**
    *   `numberOfAnalystOpinions`
    *   `recommendationMean`
    *    `recommendationKey`

### market_sentiment
*   **short_interest**
    *   `shortRatio`
    *   `sharesShort`
    *   `shortPercentOfFloat`
    *   `sharesShortPriorMonth`
    *   `dateShortInterest`
*  **enterprise_value**
    * `marketCap`
    * `enterpriseValue`
*   **ownership**
    *   `heldPercentInstitutions`
    *   `heldPercentInsiders`

### management
*   **executives**
    *    `companyOfficers`
*   **risk_factors**
    *   `auditRisk`
    *    `boardRisk`
    *    `compensationRisk`
    *    `shareHolderRightsRisk`
    *   `overallRisk`
*   **additional_info**
    *   `fullTimeEmployees`

## 7. 참고사항

-   모든 API 요청은 `https://swifin.app/v1`을 기본 URL로 사용합니다.
-   모든 API 요청은 Bearer 토큰 인증을 필요로 합니다.
-   대부분의 API 엔드포인트에서는 대문자로 티커 심볼을 입력해야 합니다.
-   페이지네이션이 필요한 API 엔드포인트는 `skip`과 `limit` 쿼리 파라미터를 사용합니다.
-   API의 사용량 제한을 확인하여 요청을 관리해야 합니다.

본 문서는 지속적으로 업데이트될 예정입니다. 궁금한 점이나 개선할 부분이 있다면 언제든지 문의해주시기 바랍니다.
