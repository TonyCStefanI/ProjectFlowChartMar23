---
layout: default
title: Flowcharts
---
## ASP.NET Core + React Gaming Platform Architecture

```mermaid
flowchart TD
    subgraph "Frontend React SPA"
        User([User])
        User --> ReactApp[React Application]
        
        subgraph "React Components"
            Pages[Pages\nHome/Games/Profile]
            GamePages[Game Pages\nBingo/Keno/Lottery]
            CommonComponents[Common Components\nButton/Modal/etc]
            LayoutComponents[Layout Components\nHeader/Footer/etc]
        end
        
        subgraph "State Management"
            AuthContext[AuthContext\nUser Session/JWT]
            GameContext[GameContext\nGame States & Logic]
            SettingsContext[SettingsContext\nLanguage/Theme/Preferences]
        end
        
        subgraph "Services & Hooks"
            ApiService[api.js\nREST API Calls]
            SignalRService[signalr.js\nReal-Time Services]
            useSignalR[useSignalR.js\nConnection Hook]
            useTranslation[useTranslation.js\nMulti-language]
        end
        
        ReactApp --> Pages
        ReactApp --> GamePages
        Pages --> CommonComponents
        Pages --> LayoutComponents
        GamePages --> CommonComponents
        GamePages --> LayoutComponents
        
        ReactApp --> AuthContext
        ReactApp --> GameContext
        ReactApp --> SettingsContext
        
        GamePages --> useSignalR
        useSignalR --> SignalRService
        Pages --> ApiService
        GamePages --> ApiService
    end
    
    subgraph "ASP.NET Core Backend"
        subgraph "API Controllers"
            AuthController[AuthController\nLogin/Register]
            GameController[GameController\nGame Catalog]
            BingoApiController[BingoApiController]
            KenoApiController[KenoApiController]
            LotteryApiController[LotteryApiController]
            BankingController[BankingController\nDeposits/Withdrawals]
        end
        
        subgraph "SignalR Hubs"
            BingoHub[BingoHub\nReal-time Bingo]
            KenoHub[KenoHub\nReal-time Keno]
            LotteryHub[LotteryHub\nReal-time Lottery]
        end
        
        subgraph "Game Engines"
            BingoGameEngine[BingoGameEngine\nServer-side Logic]
            KenoGameEngine[KenoGameEngine\nServer-side Logic]
            LotteryGameEngine[LotteryGameEngine\nServer-side Logic]
            DropPanGameEngine[DropPanGameEngine\nJamaican Lottery]
        end
        
        subgraph "Services & Data"
            AuthService[AuthService]
            GameService[GameService]
            BankingService[BankingService]
            Database[(SQL Database)]
        end
        
        BingoApiController --> BingoGameEngine
        KenoApiController --> KenoGameEngine
        LotteryApiController --> LotteryGameEngine
        
        BingoGameEngine --> BingoHub
        KenoGameEngine --> KenoHub
        LotteryGameEngine --> LotteryHub
        
        AuthController --> AuthService
        GameController --> GameService
        BankingController --> BankingService
        
        AuthService --> Database
        GameService --> Database
        BankingService --> Database
        BingoGameEngine --> Database
        KenoGameEngine --> Database
        LotteryGameEngine --> Database
    end
    
    %% API Communication
    ApiService -- "REST API\nJWT Auth" --> AuthController
    ApiService -- "REST API\nJWT Auth" --> GameController
    ApiService -- "REST API\nJWT Auth" --> BingoApiController
    ApiService -- "REST API\nJWT Auth" --> KenoApiController
    ApiService -- "REST API\nJWT Auth" --> LotteryApiController
    ApiService -- "REST API\nJWT Auth" --> BankingController
    
    %% SignalR Communication
    SignalRService -- "WebSocket\nReal-time Connection" --> BingoHub
    SignalRService -- "WebSocket\nReal-time Connection" --> KenoHub
    SignalRService -- "WebSocket\nReal-time Connection" --> LotteryHub
    
    %% Real-Time Event Flow for Bingo
    classDef frontend fill:#e1f5fe,stroke:#03a9f4,stroke-width:1px
    classDef backend fill:#e8f5e9,stroke:#4caf50,stroke-width:1px
    classDef database fill:#f3e5f5,stroke:#9c27b0,stroke-width:1px
    classDef realtime fill:#fffde7,stroke:#fbc02d,stroke-width:2px,stroke-dasharray: 5 5
    
    class ReactApp,Pages,GamePages,CommonComponents,LayoutComponents,AuthContext,GameContext,SettingsContext,ApiService,SignalRService,useSignalR,useTranslation frontend
    class AuthController,GameController,BingoApiController,KenoApiController,LotteryApiController,BankingController,BingoGameEngine,KenoGameEngine,LotteryGameEngine,DropPanGameEngine,AuthService,GameService,BankingService backend
    class Database database
    class BingoHub,KenoHub,LotteryHub,SignalRService realtime

## SignalR Real-Time Communication Flow (Bingo Example)

```mermaid
sequenceDiagram
    participant User as Player
    participant BingoGame as BingoGame.jsx
    participant useSignalR as useSignalR Hook
    participant signalr as signalR.js Service
    participant BingoHub as BingoHub
    participant Engine as BingoGameEngine
    participant DB as Database
    
    Note over User,DB: Initial Game Join
    
    User->>BingoGame: Select Bingo Room
    BingoGame->>+signalr: startConnection()
    signalr-->>-BingoGame: isConnected = true
    
    BingoGame->>+useSignalR: Subscribe to events
    useSignalR-->>-BingoGame: Setup handlers for<br>NumberCalled, WinnerFound, etc.
    
    BingoGame->>signalr: invoke('JoinGame', roomId)
    signalr->>BingoHub: JoinGame(roomId)
    BingoHub->>BingoHub: Add connection to room group
    
    Note over User,DB: REST API Call to Join Game
    
    BingoGame->>BingoApiController: POST /api/games/bingo/join
    BingoApiController->>Engine: JoinGame(userId, roomId)
    Engine->>DB: Save user's game participation
    DB-->>Engine: Success
    Engine-->>BingoApiController: Game joined successfully
    BingoApiController-->>BingoGame: 200 OK with game state
    
    Note over User,DB: Buying Bingo Cards
    
    User->>BingoGame: Buy Bingo cards
    BingoGame->>BingoApiController: POST /api/games/bingo/buy-cards
    BingoApiController->>Engine: PurchaseCards(userId, roomId, quantity)
    Engine->>DB: Deduct balance, generate cards
    DB-->>Engine: Cards generated
    Engine-->>BingoApiController: Cards purchased
    BingoApiController-->>BingoGame: 200 OK with card data
    
    Note over User,DB: Real-Time Game Play
    
    Engine->>Engine: Call next number
    Engine->>BingoHub: BroadcastNumber(roomId, number)
    BingoHub->>signalr: SendAsync("NumberCalled", number) to room group
    signalr->>useSignalR: Event "NumberCalled"
    useSignalR->>BingoGame: onNumberCalled(number)
    BingoGame->>BingoGame: Update UI with called number
    BingoGame-->>User: See called number highlighted
    
    Note over User,DB: Winner Detection
    
    User->>BingoGame: Get Bingo pattern
    BingoGame->>BingoApiController: POST /api/games/bingo/claim-win
    BingoApiController->>Engine: VerifyWin(userId, roomId, cardId, pattern)
    Engine->>Engine: Validate winning pattern
    Engine->>BingoHub: BroadcastWinner(roomId, userId, prizeAmount)
    BingoHub->>signalr: SendAsync("WinnerFound", winnerInfo) to room group
    signalr->>useSignalR: Event "WinnerFound"
    useSignalR->>BingoGame: onWinnerFound(winnerInfo)
    BingoGame->>BingoGame: Show winner message
    BingoGame-->>User: See winner notification
    
    Note over User,DB: Disconnection Handling
    
    User->>BingoGame: Leave game page
    BingoGame->>signalr: invoke('LeaveGame', roomId)
    signalr->>BingoHub: LeaveGame(roomId)
    BingoHub->>BingoHub: Remove connection from room group
    BingoGame->>useSignalR: Cleanup (componentWillUnmount)
    useSignalR->>signalr: Unsubscribe event handlers
    BingoGame->>signalr: stopConnection()

## React Frontend Component Structure with SignalR Integration

```mermaid
flowchart TD
    subgraph "App Component Structure"
        App[App.js]
        Router[React Router]
        AppProviders[AppProviders.jsx\nWraps all contexts]
        
        Layout[Layout Component]
        Header[Header Component]
        Navigation[Navigation Component]
        Footer[Footer Component]
        
        HomePage[Home Page]
        GamesPage[Games Catalog Page]
        GameDetailPage[Game Detail Page]
        
        BingoGame[BingoGame Component]
        KenoGame[KenoGame Component]
        LotteryGame[LotteryGame Component]
        
        App --> Router
        App --> AppProviders
        Router --> Layout
        
        Layout --> Header
        Layout --> Navigation
        Layout --> Content{{Content Area}}
        Layout --> Footer
        
        Content --> HomePage
        Content --> GamesPage
        Content --> GameDetailPage
        Content --> BingoGame
        Content --> KenoGame
        Content --> LotteryGame
    end
    
    subgraph "Context Providers"
        AuthCtx[AuthContext\nUser Session/JWT]
        GameCtx[GameContext\nGame States]
        SettingsCtx[SettingsContext\nPreferences]
        
        AppProviders --> AuthCtx
        AppProviders --> GameCtx
        AppProviders --> SettingsCtx
    end
    
    subgraph "Services"
        API[api.js]
        Auth[auth.js]
        SignalR[signalr.js]
        
        SignalR --> BingoSignalR[bingoSignalR\ninstance]
        SignalR --> KenoSignalR[kenoSignalR\ninstance]
        SignalR --> LotterySignalR[lotterySignalR\ninstance]
    end
    
    subgraph "Custom Hooks"
        useSignalRHook[useSignalR.js]
        useTranslationHook[useTranslation.js]
        useBreakpointHook[useBreakpoint.js]
    end
    
    %% Component connections
    BingoGame --> useSignalRHook
    KenoGame --> useSignalRHook
    LotteryGame --> useSignalRHook
    
    useSignalRHook --> BingoSignalR
    useSignalRHook --> KenoSignalR
    useSignalRHook --> LotterySignalR
    
    %% State access
    BingoGame --> GameCtx
    KenoGame --> GameCtx
    LotteryGame --> GameCtx
    
    Header --> AuthCtx
    GameDetailPage --> GameCtx
    
    %% API calls
    BingoGame --> API
    KenoGame --> API
    LotteryGame --> API
    GameDetailPage --> API
    GamesPage --> API
    
    %% Styling
    style BingoGame fill:#e3f2fd,stroke:#1976d2
    style KenoGame fill:#e3f2fd,stroke:#1976d2
    style LotteryGame fill:#e3f2fd,stroke:#1976d2
    
    style useSignalRHook fill:#fff8e1,stroke:#ffa000,stroke-width:2px
    style BingoSignalR fill:#fff8e1,stroke:#ffa000,stroke-width:2px
    style KenoSignalR fill:#fff8e1,stroke:#ffa000,stroke-width:2px
    style LotterySignalR fill:#fff8e1,stroke:#ffa000,stroke-width:2px
    
    %% Data Flow for Bingo Example
    subgraph "Bingo Real-Time Flow"
        direction TB
        BingoGameComponent[BingoGame.jsx]
        
        subgraph "Component Mount"
            UseEffect["useEffect()\nOn Component Mount"]
            JoinRoom["invoke('JoinGame', roomId)"]
            SubscribeEvents["Subscribe to events:\n- NumberCalled\n- WinnerFound\n- GameStateChanged"]
        end
        
        subgraph "SignalR Event Handling"
            NumberCalled["onNumberCalled(number)\nUpdate UI"]
            WinnerFound["onWinnerFound(winnerInfo)\nShow Winner"]
            GameStateChanged["onGameStateChanged(state)\nUpdate Game State"]
        end
        
        subgraph "Component Unmount"
            Cleanup["useEffect() cleanup\nLeaveGame + Unsubscribe"]
        end
        
        BingoGameComponent --> UseEffect
        UseEffect --> JoinRoom
        UseEffect --> SubscribeEvents
        
        SubscribeEvents --> NumberCalled
        SubscribeEvents --> WinnerFound
        SubscribeEvents --> GameStateChanged
        
        BingoGameComponent --> Cleanup
    end
    
    BingoGame --- BingoGameComponent
