
**Instruções de uso da extensão Mermaid**

Para criar, editar ou visualizar um diagrama, use as ferramentas
e comandos da extensão Mermaid para VS Code descritos abaixo.

## Fluxo de trabalho

1. Identificar o tipo de diagrama e gerar a sintaxe Mermaid correspondente.
2. Salvar o diagrama em um arquivo `.mmd` no projeto.
3. Validar a sintaxe: palavra-chave inicial correta, tipos de setas, parênteses/colchetes balanceados.
4. Pré-visualizar com a extensão Mermaid — abra o arquivo `.mmd` (pré-visualização automática) ou execute
   **Mermaid: Preview Diagram** (`mermaidChart.preview`).

## Ferramentas LLM — chamar em cada interação com diagramas

- `mermaid-diagram-validator` — valida a sintaxe Mermaid antes de apresentar o diagrama
- `mermaid-diagram-preview` — renderiza uma pré-visualização ao vivo no VS Code após gerar o diagrama
- `get-syntax-docs-mermaid` — obtém a documentação de sintaxe correta para o tipo de diagrama

## Comandos do VS Code

Use via Command Palette ou pela API de comandos do VS Code (GitHub Copilot no VS Code).
Não crie novos IDs de comando. Prefira editar arquivos `.mmd` quando um comando não for necessário.

### Edição e pré-visualização
- **Preview** (`mermaidChart.preview`) — pré-visualiza o editor Mermaid ativo (`.mmd` / `.mermaid` precisa estar aberto).
- **Create Diagram** (`mermaidChart.createMermaidFile`) — cria um fluxograma de exemplo e abre a pré-visualização lado a lado.
- **Repair Diagram** (`mermaidChart.repairDiagram`) — reparo via Mermaid AI para o diagrama ativo; consome créditos Mermaid AI.
- **Improve Diagram** (`mermaidChart.improveDiagram`) — usa Copilot / API LLM para sugerir variantes de layout e estilo.

### Gerar diagramas (requer GitHub Copilot)
- **Generate Diagram from Code** (`mermaidChart.generateDiagramFromCode`)
- **Generate Cloud Diagram** (`mermaidChart.generateCloudDiagram`)
- **Generate ER Diagram** (`mermaidChart.generateERDiagram`)
- **Generate Docker Diagram** (`mermaidChart.generateDockerDiagram`)
- **Open AI Chat** (`mermaidChart.openCopilotChat`)

### Mermaid cloud
- **Login** (`mermaidChart.login`) / **Logout** (`mermaidChart.logout`)
- **Connect Diagram** (`mermaidChart.connectDiagramToMermaidChart`) — vincula um diagrama local ao Mermaid.
- **Sync Diagram** (`mermaidChart.syncDiagramWithMermaid`) — apenas para diagramas já vinculados (frontmatter contém `id:`). Exemplo:
  ```yaml
  ---
  id: cbd9e9ba-a2cb-47c5-a98e-8c28a753428d
  ---
  ```

### Revisão do Mermaid Sync
Para diagramas atualizados pelo app Mermaid GitHub Sync (ou regenerados por pre-commit):
- **Review Mermaid Sync** (`mermaidChart.reviewAppCommits`) — inicia/abre o fluxo de revisão.
- **Regenerate with Mermaid AI** (`mermaidChart.regenerateDiagramWithMermaidAI`) — regenera a partir das referências de origem.
Não reescreva manualmente diagramas gerenciados por esse fluxo. Aceitar/rejeitar/diff devem ser feitos pela UI da extensão.

### Instalar / atualizar este pacote
- **Mermaid: Install AI Skills…** (`mermaidChart.installAiSkills`)

## Comandos slash `@mermaid-chart`

| Comando | Propósito |
|---|---|
| `/generate_diagram_from_code` | Gerar diagrama geral a partir de qualquer arquivo fonte |
| `/generate_execution_sequence` | Gerar diagrama de sequência a partir do fluxo do código |
| `/generate_er_diagram` | Gerar diagrama ER a partir de esquemas / modelos |
| `/generate_cloud_architecture_diagram` | Arquitetura Cloud / CI-CD |
| `/generate_docker_diagram` | Arquitetura a partir de Dockerfiles |
| `/generate_c4_topdown_architecture` | Arquitetura C4 top-down |
| `/analyze_code_ownership` | Diagrama de ownership do código |
| `/generate_dependency_diagram` | Visualização de dependências / segurança |

## Regras

1. Sempre chamar `mermaid-diagram-validator` antes de exibir um diagrama.
2. Sempre chamar `mermaid-diagram-preview` após gerar um diagrama.
3. Use `get-syntax-docs-mermaid` antes de gerar um tipo de diagrama desconhecido.
4. Prefira comandos slash `@mermaid-chart` para gerações complexas.
5. Grave diagramas em arquivos `.mmd`; nunca retorne sintaxe Mermaid não validada.
6. Avise o usuário antes de executar Repair (consumo de créditos Mermaid AI).
7. Coopere com o fluxo de Sync — não regenere manualmente diagramas gerenciados.

## Documentação

Mais comandos e recursos: https://marketplace.visualstudio.com/items?itemName=MermaidChart.vscode-mermaid-chart
