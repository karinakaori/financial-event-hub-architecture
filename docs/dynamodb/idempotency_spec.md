# Especificação DynamoDB — Tabela de Idempotência

Objetivo: armazenar chaves de idempotência para evitar processamento duplicado.

Nome sugerido da tabela: `hub_idempotency`

Modelo de chave primária:
- `PK` (Partition Key): `IDEMP#{idempotency_key}` (string)
- `SK` (Sort Key): `TRAN#{transaction_id}` (string) — opcional, útil para histórico

Atributos adicionais:
- `status` (string): `PENDING`, `PROCESSED`, `FAILED`
- `created_at` (string, ISO 8601)
- `processed_at` (string, ISO 8601)
- `attempts` (number): contador de tentativas de processamento

TTL (Time To Live):
- Campo TTL: `expires_at` (epoch seconds)
- Política recomendada: `expires_at = created_at + 30 dias` para entradas `PROCESSED`, e `+ 7 dias` para `FAILED` ou `PENDING`.

Índices secundários (opcionais):
- GSI por `status` para varredura operacional (GSI1: PK=status, SK=processed_at)

Estratégias de retry e consistência:
- Escrita inicial: put conditional (PUT if not exists) para evitar race conditions entre múltiplas requisições com a mesma `Idempotency-Key`.
- Leitura: leitura forte (ConsistentRead=true) quando verificar estado de idempotência antes de processar.
- Retry em falhas parciais: estratégia exponencial com jitter (ex: base 100ms, multiplicador 2, máximo 5 tentativas). Para erros idempotentes (conditional check failed), reavaliar estado e não re-tentar o processamento de negócio.
- Em caso de inconsistência detectada (duas entradas PROCESSING), o processo operacional deve acionar compensação manual.

Transações e atomicidade:
- Ao processar, usar transações DynamoDB (TransactWriteItems) para: criar registro de transação e atualizar estado atomically, quando aplicável.

