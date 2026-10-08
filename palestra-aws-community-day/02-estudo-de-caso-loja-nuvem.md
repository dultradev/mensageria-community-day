# O Estudo de Caso Guia: A Evolução da "TechStore Nuvem"

> [!TIP]
> Ter um **fio condutor único** que evolui ao longo da palestra é a melhor técnica de apresentação técnica. Isso evita que a plateia se perca em exemplos desconexos e faz com que cada tecnologia apareça como a solução natural para uma dor real do negócio.

---

## 🏬 A História da TechStore Nuvem

A **TechStore Nuvem** é uma empresa fictícia de comércio eletrônico. Acompanharemos sua jornada de arquitetura através de 5 fases evolutivas:

```mermaid
flowchart LR
    Fase1["Fase 1: Síncrona\n(O Caos)"] --> Fase2["Fase 2: Modo Fila\n(Amazon SQS)"]
    Fase2 --> Fase3["Fase 3: Modo Pub/Sub\n(SNS + SQS Fanout)"]
    Fase3 --> Fase4["Fase 4: Modo Log\n(Kafka / MSK)"]
    Fase4 --> Fase5["Fase 5: EDA Completa\n(Espinha Dorsal)"]
```

---

## 💥 Fase 1: A Arquitetura Síncrona Inicial (O Dia do Desastre)

No início do projeto, a equipe implementou tudo usando chamadas HTTP REST diretas entre os microsserviços:

```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Usuário (Navegador)
    participant API as API de Checkout
    participant Pagto as Serviço de Pagamento (Gateway)
    participant Estoque as Serviço de Estoque
    participant Fiscal as Serviço Fiscal (Sefaz)
    participant Email as Provedor de E-mail

    Cliente->>API: POST /checkout (Finalizar Pedido)
    activate API
    API->>Pagto: Processar Cartão (HTTP 300ms)
    Pagto-->>API: 200 OK
    API->>Estoque: Baixar Estoque (HTTP 200ms)
    Estoque-->>API: 200 OK
    API->>Fiscal: Emitir Nota Fiscal (HTTP 2.5s - Lento!)
    Fiscal-->>API: 200 OK
    API->>Email: Enviar E-mail (HTTP 1.2s)
    Email-->>API: 200 OK
    API-->>Cliente: 201 Created (Tempo Total: > 4.2 segundos)
    deactivate API
```

### O Desastre na Primeira Promoção:
1. **Latência Inaceitável:** O usuário fica mais de 4 segundos olhando para uma tela de carregamento.
2. **Falha em Cascata:** Durante a Black Friday, a API do serviço fiscal (Sefaz) ficou sobrecarregada e começou a demorar 15 segundos ou retornar `500 Internal Server Error`.
3. **Efeito:** Como a chamada era síncrona e bloqueante, todas as threads da API de Checkout ficaram presas esperando a resposta da nota fiscal. Novos clientes nem conseguiam abrir o site (`504 Gateway Timeout`).
4. **Vendas perdidas:** Milhares de carrinhos foram abandonados.

---

## 🛡️ Fase 2: Introdução do Modo Fila com Amazon SQS

Para salvar a operação, a equipe separou a recepção do pedido do seu processamento pesado através do **Amazon SQS**:

```mermaid
flowchart TD
    Cliente["Cliente (Navegador)"] -->|"1. POST /checkout"| API["API de Checkout"]
    API -->|"2. Envia Mensagem (~10ms)"| SQS[("Amazon SQS\n(Fila de Pedidos)")]
    API -.->|"3. 202 Accepted - Pedido Recebido!"| Cliente

    subgraph Processamento em Segundo Plano
        SQS -->|"4. Consome em lote"| Worker["Worker de Processamento"]
        Worker --> Pagto["Gateway Pagamentos"]
        Worker --> Estoque["Banco de Estoque"]
        Worker --> Fiscal["Serviço Fiscal"]
        Worker -.->|"5. Se falhar 5x"| DLQ[("Dead Letter Queue (DLQ)")]
    end
```

### O Que Mudou:
- A API de Checkout agora responde ao cliente em **menos de 50 milissegundos** com um status `202 Accepted` ("Recebemos seu pedido e já estamos processando!").
- Se o serviço Fiscal cair, os pedidos ficam empilhados em segurança dentro do SQS.
- O Worker consome no seu próprio ritmo, sem sobrecarregar o banco de dados.
- Mensagens com bugs são enviadas para a **DLQ** para não travar a fila.

---

## 📢 Fase 3: A Loja Cresceu — O Padrão Pub/Sub com SNS + SQS

Meses depois, a diretoria exige novas integrações sempre que um pedido é pago:
- O time de **Logística** precisa ser acionado para separar o pacote.
- O time de **Marketing** precisa enviar mensagens no WhatsApp e push no app.
- O time de **Prevenção à Fraude** precisa auditar a compra.

Se o Worker da Fase 2 tiver que chamar todos esses sistemas, voltamos a ter acoplamento! A solução é o **Padrão SNS + SQS Fanout**:

```mermaid
flowchart TD
    API["API de Checkout"] -->|"Publica Evento: 'PedidoPago'"| SNS[("Tópico Amazon SNS\n(pedido-pago)")]

    SNS -->|"Copia msg"| SQS1[("Fila SQS: Logística")]
    SNS -->|"Copia msg"| SQS2[("Fila SQS: Fiscal")]
    SNS -->|"Copia msg"| SQS3[("Fila SQS: Notificações")]

    SQS1 --> W1["Worker de Logística"]
    SQS2 --> W2["Worker de Nota Fiscal"]
    SQS3 --> W3["Worker de WhatsApp/Push"]
```

### O Que Mudou:
- O Checkout publica **uma única vez** no Amazon SNS.
- O SNS faz a cópia (*fanout*) para 3 filas SQS independentes.
- Se o serviço de WhatsApp estiver fora do ar, apenas a fila de notificações acumula; a logística e a nota fiscal continuam funcionando a todo vapor!
- Novo serviço no futuro? Basta criar uma nova fila SQS e inscrevê-la no tópico SNS, **com zero alterações no código do Checkout**.

---

## 📜 Fase 4: Escala Extrema e o Modo Log de Eventos com Kafka / MSK

A TechStore Nuvem se tornou um gigante do varejo. Surgiram novos desafios que filas tradicionais não conseguem atender:
1. **Telemetria de Navegação (*Clickstream*):** 100.000 cliques por segundo no site para recomendar produtos em tempo real.
2. **Replay Histórico:** O time de inteligência artificial criou um modelo novo de recomendação e precisa reprocessar os últimos 30 dias de interações para treinar a IA.
3. **Auditoria Contínua:** No SQS, a mensagem é destruída após o consumo. A empresa precisa de um log imutável de tudo o que aconteceu no sistema.

A equipe adota o **Apache Kafka gerenciado na AWS (Amazon MSK)**:

```mermaid
flowchart LR
    subgraph Produtores
        Site["Cliques no Site"]
        Checkout["Eventos de Compra"]
    end

    subgraph MSK["Amazon MSK (Cluster Kafka Gerenciado)"]
        Log[("Tópico de Log Imutável\nPartição 0: [e0][e1][e2][e3]...\nPartição 1: [e0][e1][e2]...")]
    end

    subgraph Consumidores["Consumidores com Offsets Independentes"]
        IA["Modelo de IA\n(Offset = 0: relendo histórico)"]
        Fraude["Detecção de Fraude em Tempo Real\n(Offset = 3: ponta do log)"]
        DataLake["Carga para S3 / Data Lake\n(Offset = 2)"]
    end

    Site --> Log
    Checkout --> Log

    Log -.-> IA
    Log -.-> Fraude
    Log -.-> DataLake
```

### O Que Mudou:
- As mensagens não são deletadas após o consumo. Elas são gravadas sequencialmente em um log append-only com retenção de 30 dias.
- Cada consumidor mantém seu próprio ponteiro (**Offset**).
- O time de IA pode reiniciar seu offset para zero e "viajar no tempo" (*replay*), reprocessando a história sem interferir no sistema de detecção de fraudes em produção.

---

## 🌐 Fase 5: A Visão Completa — A Mensageria como Espinha Dorsal da EDA

Chegamos à conclusão da nossa história. A TechStore Nuvem agora opera uma **Arquitetura Orientada a Eventos (EDA)** completa, onde a mensageria atua como o **sistema nervoso central** da organização:

```mermaid
flowchart TD
    subgraph Espinha["Espinha Dorsal de Eventos e Mensageria (SNS, SQS, MSK)"]
        Bus[("Barramento de Mensageria e Event Streaming")]
    end

    S1["Serviço de Vendas"] -->|"Emite 'PedidoIniciado'"| Bus
    S2["Serviço de Pagamentos"] -->|"Emite 'PagamentoAprovado'"| Bus
    
    Bus -->|"Consome evento"| S3["Serviço de Logística"]
    Bus -->|"Consome evento"| S4["Serviço Fiscal"]
    Bus -->|"Consome evento"| S5["Serviço de Notificações"]
    Bus -->|"Consome evento"| S6["Analytics & Data Lake"]

    style Bus fill:#d3adf7,stroke:#531dab,stroke-width:3px
```

### O Resumo da Conquista:
- Nenhum serviço chama o outro de forma síncrona para tarefas de negócio que toleram processamento assíncrono.
- A empresa ganha **agilidade de negócio**: novas ideias viram novos microsserviços apenas conectando-se à espinha dorsal existente.
- A estabilidade operacional e a experiência do cliente final foram blindadas contra quedas de dependências externas.
