# Matriz de Cobertura de Testes — Hub de Eventos Financeiros

Objetivo: definir critérios de aceite e cenários de teste (unitários e de integração) para o fluxo crítico de ingestão e sincronização.

Taxonomia de testes:
- Unitários: validação de validação de payload, regras de idempotência, transformação JSON→Copybook, geração de mensagens SQS.
- Integração: API → Worker → DynamoDB → SQS → Sync Worker → Mainframe (simulado via mock)
- E2E (opcional): ambiente com recursos reais ou testcontainers para DynamoDB local e SQS emulado.

Cenários críticos e critérios de aceite:

1) Requisição válida primeira vez
- Tipo: Integração
- Passos: POST válido com Idempotency-Key nova
- Aceite: resposta 202; registro `PENDING` criado no DynamoDB
- Mensagem publicada em SQS 
- Worker de sync consome e envia para mock Mainframe; entrada marcada `PROCESSED`.

2) Requisição duplicada (mesma Idempotency-Key)
- Tipo: Unitário + Integração
- Passos: repetir POST com mesma Idempotency-Key
- Aceite: API retorna 409 ou 200 com referência ao estado
- Nenhum novo processamento duplicado é executado
- DynamoDB mantém estado original

3) Falha parcial (Worker publica na fila, mas falha antes de confirmar)
- Tipo: Integração
- Passos: simular falha após Put em DynamoDB mas antes de SendMessage
- Aceite: mecanismo de retry detecta falha e re-tenta publicação
- Se atingir limite, DLQ recebe mensagem
- Estado do idempotency reflete tentativa falhada com `attempts` incrementado

4) Timeout do Mainframe / MQ
- Tipo: Integração
- Passos: Sync Worker envia e aguarda ACK
- Simular timeout do mock Mainframe
- Aceite: Sync Worker re-tenta conforme política (exponencial com jitter)
- Após 3 tentativas, move mensagem para DLQ
- Estado manual/operacional requerido para reconciliar

5) Mensagem malformada na fila
- Tipo: Unitário + Integração
- Passos: enviar payload que não corresponde ao JSON Schema
- Aceite: consumidor descarta ou envia para DLQ com motivo
- Logs e métricas registram causa

6) Teste de carga básica
- Tipo: Integração / Performance
- Passos: enviar N requests por segundo (ex: 100rps) por 5 minutos
- Aceite: sistema processa com latência aceitável (p.ex. API < 300ms para accept), fila cresce mas não perde mensagens
- Não há perda de dados.

Cenarios de borda e observabilidade:
- Métricas e logs necessários: taxa de input, latência das operações DynamoDB, taxa de consumos SQS, taxas de falhas e tamanho da DLQ.
- Alertas: aumento súbito na DLQ, aumento de latência ou erros 5xx.

