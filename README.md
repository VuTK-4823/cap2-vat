%% Sơ đồ C&C (Component & Connector) cho Module An toàn
graph TD
    subgraph "Bên ngoài (External)"
        CLIENT[Client Browser]
        MAIN_SYS[Main System (Java)]
        EXT_SEARCH[External: Google Search API]
        EXT_AI[External: Google Gemini API]
    end

    subgraph "Module An toàn (Node.js)"
        API_SERVER[<b>Component: API Server</b><br>(apiServer.js)]
        SCORING_JOB[<b>Component: Scoring Job</b><br>(runJob)]
        AI_WORKER[<b>Component: AI Worker</b><br>(aiWorker.js)]
    end

    subgraph "CSDL An toàn (PostgreSQL)"
        DB_INPUTS[<b>Data: Core Data Tables</b><br>(properties, reviews, incidents, ...)]
        DB_QUEUE[<b>Data: Job Queue Table</b><br>(ai_generation_queue)]
        DB_RESULT[<b>Data: Result Table</b><br>(property_safety_scores)]
    end

    %% --- Connectors ---
    
    %% Client Connectors
    CLIENT -- "<b>Connector: HTTP API Call</b><br>(GET /safety, POST /review)" --> API_SERVER
    API_SERVER -- "<b>Connector: HTTP API Call</b><br>(Trả về JSON/HTML)" --> CLIENT
    
    %% Main System Connectors
    MAIN_SYS -- "<b>Connector: HTTP API Call</b><br>(POST /sync)" --> API_SERVER
    
    %% API Server Connectors
    API_SERVER -- "<b>Connector: DB Query</b><br>(Ghi Review/Sự cố)" --> DB_INPUTS
    API_SERVER -- "<b>Connector: DB Query</b><br>(Đọc Kết quả)" --> DB_RESULT
    API_SERVER -- "<b>Connector: Job Trigger</b><br>(Kích hoạt (Async))" --> SCORING_JOB
    
    %% Scoring Job Connectors
    SCORING_JOB -- "<b>Connector: DB Query</b><br>(Đọc 3 trụ cột dữ liệu)" --> DB_INPUTS
    SCORING_JOB -- "<b>Connector: DB Query</b><br>(Ghi Điểm số)" --> DB_RESULT
    SCORING_JOB -- "<b>Connector: Queue Message</b><br>(Ghi 'pending' job)" --> DB_QUEUE
    
    %% AI Worker Connectors
    AI_WORKER -- "<b>Connector: Queue Message</b><br>(Đọc 'pending' job)" --> DB_QUEUE
    AI_WORKER -- "<b>Connector: HTTP API Call</b><br>(Gọi AI/Search)" --> EXT_SEARCH
    AI_WORKER -- "<b>Connector: HTTP API Call</b><br>(Gọi AI/Search)" --> EXT_AI
    AI_WORKER -- "<b>Connector: DB Query</b><br>(Ghi 'ai_summary')" --> DB_RESULT
    AI_WORKER -- "<b>Connector: Queue Message</b><br>(Ghi 'done' job)" --> DB_QUEUE
