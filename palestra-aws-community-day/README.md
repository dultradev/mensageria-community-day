# 🎙️ Preparação para a Palestra: AWS Community Day

> **Tema:** Primeiros Passos em Mensageria: SQS, SNS, Kafka e o Mundo dos Eventos  
> **Formato:** Apresentação em trio (3 palestrantes)  
> **Duração Estimada:** 40 a 45 minutos (+ 5 minutos para Q&A da plateia)  
> **Nível:** Introdutório / Intermediário (foco conceitual com exemplos práticos e arquitetura na AWS)

---

## 🎯 Objetivo da Palestra

Desmistificar o universo de mensageria para a comunidade de desenvolvedores e arquitetos, conduzindo a plateia por uma jornada evolutiva: desde a dor das integrações síncronas tradicionais até a maturidade das Arquiteturas Orientadas a Eventos (EDA), contextualizando quando utilizar **Amazon SQS**, **Amazon SNS** e **Apache Kafka / Amazon MSK**.

---

## 🗺️ Mapa da Jornada da Apresentação

```mermaid
flowchart LR
    A["1. O Problema\n(Síncrono quebrando)"] --> B["2. A Resposta\n(Mensageria & Conceitos)"]
    B --> C["3. Modo Fila\n(Amazon SQS)"]
    C --> D["4. Modo Pub/Sub\n(Amazon SNS)"]
    D --> E["5. Modo Log de Eventos\n(Kafka / Amazon MSK)"]
    E --> F["6. A Grande Conexão\n(Mensageria como Espinha Dorsal da EDA)"]

    style A fill:#ff4d4f,stroke:#333,color:#fff
    style B fill:#ffa39e,stroke:#333
    style C fill:#ffe58f,stroke:#333
    style D fill:#d9f7be,stroke:#333
    style E fill:#bae7ff,stroke:#333
    style F fill:#d3adf7,stroke:#333
```

---

## 👥 Estratégia de Divisão entre os 3 Palestrantes

Para que a apresentação em trio seja dinâmica e não pareça três palestras desconexas, cada palestrante assume um ato específico de uma **história contínua** (o caso da *TechStore Nuvem*):

| Bloco | Palestrante | Tempo Aprox. | Foco Temático e Serviços |
| :---: | :---: | :---: | :--- |
| **Ato 1** | **Palestrante 1** | ~12 min | **A Dor Síncrona e o Despertar da Mensageria:**<br>• O exemplo do e-commerce quebrando na promoção (HTTP em cascata, timeout, cascading failures).<br>• O que é mensageria e seus benefícios (desacoplamento, resiliência, buffering).<br>• Anatomia essencial da mensagem e os atores (Produtor, Broker, Consumidor). |
| **Ato 2** | **Palestrante 2** | ~15 min | **O Modo Fila e o Modo Pub/Sub na AWS:**<br>• Modo Fila com **Amazon SQS** (Standard vs FIFO, Visibility Timeout, DLQ).<br>• Modo Pub/Sub com **Amazon SNS** (Tópicos, Fanout SNS $\rightarrow$ SQS, Filtros).<br>• Resolução prática do problema da loja com SQS e SNS. |
| **Ato 3** | **Palestrante 3** | ~13 min | **O Log de Eventos e a Visão de Futuro (EDA):**<br>• Por que filas não resolvem tudo? A necessidade de histórico, replay e streaming.<br>• Modo Log de Eventos com **Apache Kafka e Amazon MSK** (Partições, Offsets, imutabilidade).<br>• Introdução à EDA: Mensageria como a espinha dorsal dos sistemas modernos.<br>• Fechamento, dicas finais e chamada para Q&A. |

---

## 📂 Materiais Deste Kit de Preparação

1. [**01. Roteiro Slide a Slide e Transições**](./01-roteiro-e-slides.md)  
   Estrutura de 20 slides pronta para montagem da apresentação, com pontos de fala (*talking points*), ganchos de passagem de bastão entre os palestrantes e armadilhas a evitar no palco.

2. [**02. O Estudo de Caso Guia: TechStore Nuvem**](./02-estudo-de-caso-loja-nuvem.md)  
   O exemplo visual unificado do e-commerce que evolui ao longo dos slides para prender a atenção da plateia do início ao fim.

3. [**03. Guia Prático dos Serviços: SQS, SNS e Kafka (MSK)**](./03-guia-aws-sqs-sns-msk.md)  
   Foco aprofundado nos três serviços do título da palestra, comparativos lado a lado e quando escolher cada um.

4. [**04. Analogias Memoráveis de Palco e FAQ da Plateia**](./04-analogias-e-faq-plateia.md)  
   Metáforas fáceis de assimilar para o palco e preparação com respostas prontas para perguntas difíceis que a plateia do AWS Community Day costuma fazer.
