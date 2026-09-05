# Hub de Eventos Financeiros - Documentação & Diagramas como Código

## 1. Visão Geral do Sistema
- **Domínio e Escopo:** Plataforma responsável por capturar eventos transacionais da AWS e sincronizar de forma assíncrona com o Core Bancário Legado (Mainframe/COBOL via IBM MQ).
- **Nível da Visão:** C4 Model (Containers) e Diagrama de Sequência.
- **Componentes:** AWS API Gateway, Ingestion Worker (Java/Spring Boot), DynamoDB (Idempotência), AWS SQS, Mainframe Sync Worker (Python) e Core Legado (COBOL/MQ).
- **Restrições:** Baixa latência, garantia *At-Least-Once*, compliance com LGPD/PCI-DSS e idempotência estrita.

## 2. Diagramas em Mermaid.js

### A. Diagrama Estrutural: Visão de Containers (C4 Model - Nível 2)
```mermaid
C4Container
    title Diagrama de Containers - Hub de Eventos Financeiros

    Person(customer, "Cliente/App", "Realiza transações financeiras.")
    
    System_Boundary(c1, "Financial Event Hub") {
        Container(api_gw, "Gateway de API", "AWS API Gateway", "Autenticação, rate limiting e roteamento")
        Container(tx_worker, "Worker de Ingestão de Transações", "Java / Spring Boot", "Valida payloads e garante idempotência no DynamoDB")
        ContainerDb(idempotency_db, "Armazenamento de Idempotência", "DynamoDB", "Armazena hash de transações e chave de deduplicação")
        Container(event_queue, "Fila de Transações", "AWS SQS", "Fila principal de eventos com DLQ acoplada")
        Container(sync_service, "Worker de Sincronização com Mainframe", "Python / AsyncIO", "Consome fila e formata registros para o protocolo Mainframe")
    }

    SystemDb_Ext(legacy_mainframe, "Core Bancário Legado", "IBM Mainframe / COBOL (MQ)", "Sistema de registro final de contas e saldos")

    Rel(customer, api_gw, "Envia transação HTTP/REST", "JSON/HTTPS")
    Rel(api_gw, tx_worker, "Encaminha chamada", "gRPC / Internal REST")
    Rel(tx_worker, idempotency_db, "Verifica/Registra Idempotência", "DynamoDB SDK")
    Rel(tx_worker, event_queue, "Publica evento processado", "AWS SDK SQS")
    Rel(sync_service, event_queue, "Consome mensagens", "Long Polling")
    Rel(sync_service, legacy_mainframe, "Sincroniza registro", "IBM MQ / EBCDIC/Fixed-Width")
```

### B. Diagrama de Sequência
```mermaid
sequenceDiagram
    autonumber
    actor Client as Cliente/App
    participant API as Gateway de API
    participant Worker as Worker de Ingestão (Java)
    participant Dynamo as DynamoDB (Idempotência)
    participant SQS as Fila SQS
    participant Sync as Worker de Sincronização (Python)
    participant Mainframe as Core Legado (COBOL/MQ)

    Client->>API: POST /v1/transactions (Payload + Idempotency-Key)
    API->>Worker: Repassa Requisição
    Worker->>Dynamo: Check & Put(Idempotency-Key)
    
    alt Chave Já Existe (Duplicado)
        Dynamo-->>Worker: Status: PENDING / PROCESSED
        Worker-->>API: HTTP 409 / 200 (Retorna estado salvo)
        API-->>Client: Resposta Cacheada
    else Chave Nova
        Dynamo-->>Worker: OK (Lock Adquirido)
        Worker->>SQS: SendMessage(TransactionEvent)
        SQS-->>Worker: MessageId Confirmado
        Worker-->>API: HTTP 202 Accepted (TransactionId)
        API-->>Client: Confirmado Processamento Assíncrono
        
        loop Worker Assíncrono
            Sync->>SQS: ReceiveMessages()
            SQS-->>Sync: Event Payload
            Sync->>Mainframe: Send Copy (Copybook EBCDIC via MQ)
            Mainframe-->>Sync: ACK / Sync Success
            Sync->>SQS: DeleteMessage()
        end
    end
```
