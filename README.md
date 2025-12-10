Exemplo de Arquitetura Resiliente com Mensageria na Cloud
Outbox Pattern • Azure Service Bus • Workers • Resiliência com Polly • Demonstração Didática para PosTech Arquitetura de Sistemas .NET

Este projeto demonstra uma arquitetura moderna e resiliente baseada em mensageria, utilizando:

API REST .NET 8 para receber solicitações de crédito

Outbox Pattern para garantir consistência e evitar duplicidade

Azure Service Bus como barramento de mensagens

Workers .NET 8 para:

Publicação assíncrona (com Polly, Circuit Breaker e Retry com Backoff)

Consumo de mensagens (message pump)

Expurgo automático da tabela Outbox

SQL Server em Docker persistindo:

Outbox

Propostas (créditos aprovados)

Solicitações Rejeitadas

Esse exemplo foi projetado para fins educacionais e apresenta conceitos amplamente utilizados no mercado bancário e de alta criticidade.

📌 1. Objetivo da Solução

A solução simula o fluxo real de solicitação de crédito:

A API recebe uma solicitação e salva em uma tabela Outbox, garantindo:

Idempotência

Persistência confiável antes da publicação

O worker-publicador lê a Outbox e publica para o Azure Service Bus, com:

Retry exponencial via Polly

Circuit Breaker

Log estruturado via Serilog

O worker-consumidor recebe as mensagens e processa:

Se aprovado → gera uma Proposta

Se rejeitado → registra uma Solicitação Rejeitada

O worker-expurgo-outbox limpa mensagens antigas publicadas.

🧱 2. Arquitetura da Solução

A arquitetura segue o modelo C4, com níveis 1, 2 e 3.

2.1 Contexto (C4-Nível 1)

Usuário → envia uma solicitação de crédito

API → persiste Outbox e responde imediatamente

Worker-publicador → publica no broker

Service Bus → entrega para consumidores

Worker-consumidor → gera Propostas ou Rejeições

Worker-expurgo → mantém a Outbox limpa

2.2 Containers (C4-Nível 2)

Os principais containers:

api-solicitacao-credito

worker-publicador

worker-consumidor

worker-expurgo-outbox

sqlserver

azure service bus

2.3 Componentes (C4-Nível 3)

Cada worker e a API têm diagramas de componentes:

API: Controller, Service, Idempotency, EF Core + Outbox

Publicador: OutboxReader, Publisher, Polly Resilience Layer

Consumidor: MessagePump, ProcessadorCredito, DbContexts

Expurgo: Scheduler + CleanupService

🚀 3. Fluxo Completo da Demonstração
1) API recebe solicitação

Calcula uma Idempotency Key baseada nos dados + janela de 48 horas

Salva na Outbox

Retorna HTTP 202 (Accepted)

2) Worker-publicador

Lê mensagens pendentes

Publica no Azure Service Bus

Marca como "Publicada"

3) Worker-consumidor

Ouve a fila continuamente

Processa:

Aprova crédito → cria Proposta

Rejeita crédito → cria SolicitacaoRejeitada

4) Worker-expurgo

Executa diariamente às 23h

Remove registros Outbox com status Publicada e mais antigos que X dias

🛠 4. Tecnologias Utilizadas
Componente	Tecnologia
API	.NET 8 / ASP.NET Web API
Outbox Pattern	EF Core 8
Broker	Azure Service Bus
Workers	.NET 8 BackgroundService
Resiliência	Polly (Retry + Circuit Breaker)
Logging	Serilog (Log estruturado)
Banco de Dados	SQL Server 2022 (Docker)
Orquestração	Docker Compose
