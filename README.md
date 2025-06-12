# dsa-real-time-etl-airflow-spark-kafka
Projeto Kafka + Spark + Cassandra + Airflow

# Pipeline de Processamento de Dados em Tempo Real com Kafka, Spark, Cassandra e Airflow

Este projeto implementa uma arquitetura moderna de dados em tempo real, baseada no padrão **Cliente-Servidor**, utilizando:

- Apache Kafka
- Apache Spark Structured Streaming
- Apache Cassandra
- Apache Airflow
- Docker + Docker Compose

---

## Objetivo do projeto

Criar um pipeline de ingestão e processamento de eventos de dados em tempo real, com:

✅ Governança e rastreabilidade dos dados  
✅ Automação e orquestração de pipelines  
✅ Separação clara de responsabilidades entre infraestrutura (servidor) e lógica de negócio (cliente)  
✅ Escalabilidade e resiliência  
✅ Capacidade de resposta em tempo real para o negócio  

---

## 🗺️ Arquitetura geral do projeto

### 🏗️ Modelo Cliente-Servidor

Neste projeto, a arquitetura segue um modelo **Cliente-Servidor bem definido**:

### 🔹 Servidor (Infraestrutura - containers orquestrados)

- **Kafka Broker** → gerenciamento de mensagens em tempo real
- **Schema Registry** → controle de formatos de mensagens Kafka
- **Kafka Control Center** → observabilidade da stack Kafka
- **Apache Spark Master e Worker** → cluster de processamento de dados
- **Apache Cassandra** → banco NoSQL para persistência dos dados
- **Apache Airflow** → orquestração e automação de workflows
- **Postgres** → backend de metadados para o Airflow

Todos estes componentes são orquestrados pelo `docker-compose.yml`, e formam a **plataforma de dados e processamento** do projeto.

---

### 🔸 Cliente (Lógica de processamento - container separado)

- **dsa_cliente** → container separado que executa um consumidor Kafka implementado com Spark Structured Streaming.
- Responsável por:
  - Consumir eventos do Kafka topic `dsa_kafka_topic`.
  - Processar e transformar os dados com Spark.
  - Persistir os resultados no Cassandra.
- Totalmente isolado da infraestrutura → permite escalar horizontalmente e atualizar a lógica de negócio sem impactar os serviços core.

---

## 🔍 Fluxo da arquitetura

```text
Kafka Producer (mockado) → Kafka Broker → Cliente Kafka-Spark-Cassandra (dsa_cliente) → Cassandra DB → Airflow (orquestração futura)
Rede
Todos os containers estão conectados na rede dsaservidor_dsacademy, garantindo comunicação segura e isolada.

⚙️ Tecnologias utilizadas
Apache Kafka

Apache Spark

Apache Cassandra

Apache Airflow

Docker

Docker Compose

🗂️ Estrutura do projeto
text
Copiar
Editar
/
├── dags/                             # DAGs do Airflow
├── entrypoint/                       # Entrypoint do Airflow
├── modulos/                          # Módulos auxiliares (montado como volume)
├── requirements.txt                  # Dependências Python
├── Dockerfile                        # Imagem do dsa_cliente (Kafka-Spark-Cassandra Consumer)
├── docker-compose.yml                # Orquestração de todos os containers
├── dsa_consumer_stream.py            # Kafka-Spark-Cassandra Consumer (CLIENTE)
└── README.md                         # Documentação do projeto
🚀 Execução do projeto
1️⃣ Subir os containers de infraestrutura (Servidor)
bash
Copiar
Editar
docker-compose pull
docker-compose up -d
2️⃣ Criar a imagem do dsa_cliente
bash
Copiar
Editar
docker build -t kafka-spark-cassandra-consumer .
3️⃣ Rodar o container dsa_cliente (Cliente separado)
bash
Copiar
Editar
# Exemplo modo inicial (reprocessar histórico):
docker run --name dsa_cliente --network dsaservidor_dsacademy kafka-spark-cassandra-consumer python dsa_consumer_stream.py --mode initial

# Exemplo modo append (tempo real):
docker run --name dsa_cliente --network dsaservidor_dsacademy kafka-spark-cassandra-consumer python dsa_consumer_stream.py --mode append
📝 Detalhamento do Kafka Consumer (Cliente)
O consumer dsa_consumer_stream.py:

✅ Se conecta ao tópico dsa_kafka_topic no Kafka
✅ Usa Spark Structured Streaming para consumir e transformar os dados
✅ Converte os dados em DataFrame estruturado
✅ Realiza sanitização e filtragem dos dados
✅ Insere os dados na tabela dsa_dados_usuarios.tb_usuarios no Cassandra

Schema da tabela Cassandra:
sql
Copiar
Editar
CREATE TABLE IF NOT EXISTS dsa_dados_usuarios.tb_usuarios (
    id TEXT PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    gender TEXT,
    address TEXT,
    post_code TEXT,
    email TEXT,
    username TEXT,
    dob TEXT,
    registered_date TEXT,
    phone TEXT,
    picture TEXT
);
✅ Problemas enfrentados e soluções aplicadas
Separação Cliente-Servidor
→ A arquitetura Cliente-Servidor foi adotada para garantir:

✅ Isolamento da lógica de negócio → o dsa_cliente é container separado.
✅ Escalabilidade → é possível rodar múltiplos consumidores Kafka em paralelo.
✅ Independência → alterações no código do cliente não afetam os serviços core.

Restrições de permissões no ambiente local
→ Uso de containers para isolar dependências e evitar problemas com permissões no Windows.

Comunicação entre containers
→ Rede dsaservidor_dsacademy criada no Compose → cliente explicitamente conectado à mesma rede.

Problema com entrypoint do Airflow Webserver
→ chmod removido do entrypoint → script executado diretamente.

Kafka Consumer otimizado
→ Uso de Spark Structured Streaming para garantir escalabilidade e performance.

🚀 Benefícios da arquitetura
✅ Arquitetura Cliente-Servidor escalável e modular
✅ Processamento de dados em tempo real
✅ Separação de responsabilidades entre infraestrutura e lógica de consumo
✅ Flexibilidade para evoluir novos casos de uso
✅ Modularidade e escalabilidade (consumidores Kafka isolados em containers separados)
✅ Governança e rastreabilidade dos pipelines de dados
✅ Infraestrutura como código com Docker Compose

🌍 Aplicações reais
Monitoramento de operações em tempo real

Análise de comportamento de usuários

Detecção de anomalias em processos industriais

Suporte a dashboards de inteligência operacional

Enriquecimento de dados em tempo real

📚 Aprendizados
✅ Separação de responsabilidades entre servidor e cliente é essencial para arquitetura escalável.
✅ Kafka + Spark + Cassandra é uma stack extremamente poderosa para pipelines em tempo real.
✅ Automatização e containerização reduzem drasticamente a complexidade operacional.
✅ Monitoramento, logging e governança são peças fundamentais de uma arquitetura de dados moderna.

📌 Próximos passos
Implementar batch inserts para Cassandra

Automatizar execução do dsa_cliente com DAG do Airflow

Implementar monitoramento de offsets do Kafka

Instrumentar métricas no Control Center

Validar escalabilidade com múltiplos consumers concorrentes

🤝 Contribuição
Este projeto foi implementado como parte de estudos e prática de arquitetura de dados em tempo real e automação de pipelines críticos, com o objetivo de explorar boas práticas e padrões de mercado.

Contribuições e sugestões são sempre bem-vindas!
