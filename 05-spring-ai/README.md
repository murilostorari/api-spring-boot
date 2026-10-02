# DIO Spring Boot - Final Project 05: Spring AI (Budgeting API)

## O que o projeto faz

API de orçamento que integra **Spring AI** para processar comandos de voz relacionados a transações financeiras. O fluxo principal:

1. **Recebe um arquivo de áudio** enviado pelo cliente
2. **Transcreve o áudio em texto** (Speech-to-Text via OpenAI Whisper)
3. **Usa IA para entender a intenção** e selecionar a ferramenta apropriada (Tool Calling)
4. **Executa a ação** — cria ou consulta transações financeiras
5. **Retorna a resposta em áudio** (Text-to-Speech via OpenAI TTS)

## Como executar a aplicação

### Pré-requisitos

- **Java 17+** (JDK 17 ou superior)
- **Gradle 8.x** (ou use o `./gradlew` incluído no projeto)
- **Chave da API OpenAI** — [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)

### Passo 1: Configurar a chave da API

```bash
export OPENAI_API_KEY="sua-chave-aqui"
```

No Windows (PowerShell):
```powershell
$env:OPENAI_API_KEY="sua-chave-aqui"
```

### Passo 2: Executar a aplicação

```bash
./gradlew bootRun
```

A aplicação inicia em `http://localhost:8080`

### Banco de dados

Por padrão, o projeto usa **H2 em memória** (ideal para desenvolvimento). Para usar MySQL via Docker:

```bash
docker compose up -d
```

> **Nota:** O compose.yml já está configurado para MySQL 9.6 na porta 3307.

## Endpoints da API

| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/transactions` | Cria uma transação (JSON) |
| `GET` | `/transactions/{category}` | Lista transações por categoria |
| `POST` | `/transactions/ai` | Envia áudio → IA processa → retorna áudio |

### Exemplo: Criar transação via REST

```bash
curl -X POST http://localhost:8080/transactions \
  -H "Content-Type: application/json" \
  -d '{"description":"Compras no mercado","category":"GROCERIES","amount":15000}'
```

### Exemplo: Listar transações por categoria

```bash
curl http://localhost:8080/transactions/GROCERIES
```

### Exemplo: Enviar áudio para a IA

```bash
curl -X POST http://localhost:8080/transactions/ai \
  -F "file=@meu-audio.m4a" \
  --output resposta.mp3
```

## Estrutura do projeto (DDD)

```
src/main/java/dio/budgeting/
├── BudgetingApplication.java          # Classe principal
├── domain/                             # Domínio (núcleo da aplicação)
│   ├── Category.java                   # Enum de categorias
│   ├── Transaction.java                # Entidade de domínio
│   ├── TransactionId.java            # Identificador tipado
│   └── TransactionRepository.java      # Contrato do repositório
├── application/                        # Casos de uso
│   ├── input/
│   │   └── PersistTransactionInput.java
│   ├── output/
│   │   └── TransactionOutput.java
│   ├── PersistTransactionUseCase.java  # Cria transação (Tool)
│   └── ListTransactionsByCategoryUseCase.java  # Lista por categoria (Tool)
└── infrastructure/                     # Adaptadores
    ├── http/
    │   ├── TransactionController.java  # REST + AI endpoints
    │   ├── request/TransactionRequest.java
    │   └── response/TransactionResponse.java
    └── persistence/
        ├── entity/TransactionEntity.java
        ├── repository/JpaTransactionRepository.java  # Adapter do repositório
        └── repository/TransactionEntityRepository.java  # Spring Data
```

## Tecnologias usadas

| Tecnologia | Versão | Uso |
|------------|--------|-----|
| Spring Boot | 3.3.x | Framework principal |
| Spring AI | 1.0.x | Integração com modelos de IA |
| OpenAI | API | Whisper (transcrição), GPT (chat), TTS (voz) |
| Spring Data JPA | — | Persistência |
| H2 | — | Banco em memória (dev) |
| MySQL | 9.6 | Banco em produção (Docker) |
| Lombok | — | Redução de boilerplate |
| Gradle | 8.10 | Build tool |

## Como testar

```bash
./gradlew test
```

Os testes de integração (`IT`) que usam a API OpenAI são pulados automaticamente se a variável `OPENAI_API_KEY` não estiver definida.

## Melhoria implementada

### 1. Banco de dados H2 em memória
O projeto base original usa MySQL via Docker Compose. Para facilitar o desenvolvimento rápido e não depender de Docker, configurei o H2 em memória como banco padrão. Isso permite:
- Executar a aplicação sem Docker
- Banco de dados reiniciado a cada execução (ideal para testes)
- Console web do H2 em `http://localhost:8080/h2-console`

### 2. Banco de dados para testes (H2)
Os testes usam um banco H2 separado (`src/test/resources/application.properties`) para isolar o ambiente de teste.

## O que aprendi

- **Tool Calling do Spring AI**: como expor métodos Java (@Tool) para que a IA os invoque automaticamente
- **Transcrição de áudio**: Speech-to-Text com Whisper via `TranscriptionModel`
- **Geração de voz**: Text-to-Speech com `TextToSpeechModel`
- **Arquitetura limpa (DDD)**: separação clara entre domínio, aplicação e infraestrutura
- **Spring Boot + Spring AI**: integração fluida entre a aplicação e modelos de linguagem

## Referências

- [Spring AI Reference](https://docs.spring.io/spring-ai/reference/index.html)
- [Repositório completo da trilha DIO](https://github.com/digitalinnovationone/dio-spring-boot-learning-track)
