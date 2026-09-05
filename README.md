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

    Person(customer, "Cliente/App")

    System_Boundary(c1, "Hub de Eventos Financeiros") {
        Container(api_gw, "Gateway API", "API HTTP")
        Container(tx_worker, "Worker de Ingestão", "Processamento síncrono")
        ContainerDb(idempotency_db, "Armazenamento de Idempotência", "DynamoDB")
        Container(event_queue, "Fila de Eventos", "SQS")
        Container(sync_service, "Worker de Sincronização", "Processamento assíncrono")
    }

    System_Ext(legacy_mainframe, "Core Legado")

    Rel(customer, api_gw, "1")
    Rel(api_gw, tx_worker, "2")
    Rel(tx_worker, idempotency_db, "3")
    Rel(tx_worker, event_queue, "4")
    Rel(sync_service, event_queue, "5")
    Rel(sync_service, legacy_mainframe, "6")
```

Legenda:

- 1: Requisição HTTP com `Idempotency-Key`
- 2: Encaminhamento para processamento interno
- 3: Verificação / gravação de idempotência (DynamoDB)
- 4: Publicação do evento processado na fila (SQS)
- 5: Consumo assíncrono pelo `Worker de Sincronização`
- 6: Envio ao Core Legado (MQ)

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
