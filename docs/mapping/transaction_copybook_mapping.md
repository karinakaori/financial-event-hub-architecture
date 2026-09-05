# Mapeamento JSON -> Copybook (COBOL)

Este arquivo define a equivalência campo-a-campo entre o JSON publicado no SQS e o layout fixo COBOL (copybook) usado pelo Core Legado.

Formato JSON de referência (SQS):
- `transaction_id` (string, UUID)
- `account_id` (string)
- `amount_cents` (integer)
- `currency` (string, 3)
- `processed_at` (string, ISO 8601)
- `idempotency_key` (string)
- `metadata` (string, JSON)

Mapeamento para Copybook (exemplo fixo, posições e tamanhos em bytes):

Copybook (COBOL) exemplo:

       01  TRANSACTION-RECORD.
           05  TRANSACTION-ID        PIC X(36).
           05  ACCOUNT-ID            PIC X(20).
           05  AMOUNT-CENTS          PIC 9(13).
           05  CURRENCY              PIC X(3).
           05  PROCESSED-AT          PIC X(26).
           05  IDEMPOTENCY-KEY       PIC X(64).
           05  METADATA              PIC X(200).

Regras de mapeamento e transformação:
- `transaction_id`: copiar string UTF-8 (UUID) diretamente para `TRANSACTION-ID`. Se menor que 36, preencher com espaços à direita.
- `account_id`: truncar/pad right até 20 caracteres; caracteres não-ASCII devem ser substituídos ou removidos conforme política de transliteração.
- `amount_cents`: número inteiro até 13 dígitos. No JSON é representado em centavos. Validar overflow antes do mapeamento.
- `currency`: truncar/upper-case para 3 caracteres.
- `processed_at`: formatar como `YYYY-MM-DDTHH:MM:SSZ` (26 chars) e mapear para `PROCESSED-AT`.
- `idempotency_key`: mapear para PIC X(64). Se a chave do cliente exceder 64 chars, gerar hash SHA-256 hex truncado para 64 chars.
- `metadata`: serializar JSON minimalmente e truncar para 200 chars.

Observações operacionais:
- O adaptador que transforma JSON → Copybook deve validar campos obrigatórios e retornar erro em caso de violação de tamanho, tipo ou falta de dados.
- A transformação deve ser determinística para facilitar logs e debugging (mesma entrada → mesma saída byte-a-byte).

