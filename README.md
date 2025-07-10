# thesis
Fault Prediction and Automated Remediation in Cloud-Native Kubernetes Environments
This repository contains the code and documentation for the thesis titled "Fault Prediction and Automated Remediation in Cloud-Native Kubernetes Environments". The thesis explores the integration of machine learning techniques with Kubernetes for proactive fault management.
## Overview
The project implements a system that collects metrics, logs, and events from Kubernetes clusters, processes this data to detect anomalies, and automates remediation actions using machine learning models. The architecture includes components for data collection, anomaly detection, and automated remediation, leveraging technologies like Prometheus, OpenTelemetry, Kafka, Redis, and Grafana.
## Architecture
The architecture diagram illustrates the flow of data and interactions between components in the system. It includes:
- **Kubernetes Cluster**: Collects metrics, logs, and events from user applications and the Kubernetes API.
- **Data Ingestion & Storage**: Uses a Go application to collect data from Prometheus and OpenTelemetry, which is then streamed to Kafka and stored in Redis.
- **Intelligent Remediation System Core Logic**: Processes the data to detect anomalies, generates Grafana dashboards, and interacts with a machine learning model to propose remediation actions.
- **Operations & Monitoring**:
Integrates with Grafana for visualization and Alert Manager for notifications.
## Components
- **Data Collector**: A Go application that collects metrics and logs from Kubernetes and streams them to Kafka.
- **Kafka Cluster**: Acts as a message broker for streaming data.
- **Redis Cluster**: Stores the streamed data for processing and aggregation.
- **Anomaly Detection**: Uses a trigger to detect anomalies in the data, processes it in parallel, and aggregates results.
- **Grafana Dashboard Generator**: Creates dashboards based on the aggregated data.
- **MCP Server**: Implements the Model Context Protocol to interact with the machine learning model and Kubernetes API for remediation actions.
- **LLM Integration**: Uses a large language model e.g., OpenAI API to analyze the context and propose actions based on the detected anomalies.
- **Feedback Loop**: Collects learning data to improve the system over time.

```mermaid
graph TD
    subgraph Kubernetes Cluster
        K8SAPI[Kubernetes API]
        PROM[Prometheus]
        OTEL[OpenTelemetry, Tempo, Loki]
        APP[User Applications/Pods]
        EVENT[Kubernetes Events]

        APP -- Metrics --> PROM
        APP -- Traces/Logs --> OTEL
        K8SAPI -- Pod/Container Metadata --> DATA_COLLECTOR
        EVENT -- Events --> DATA_COLLECTOR
    end

    subgraph Data Ingestion & Storage
        DATA_COLLECTOR[Data Collector Go Application]
        KAFKA[Kafka Cluster]
        REDIS[Redis Cluster]

        PROM -- Pull --> DATA_COLLECTOR
        OTEL -- Export --> DATA_COLLECTOR
        DATA_COLLECTOR -- podEventPayload --> KAFKA
        KAFKA -- Stream --> REDIS
    end

    subgraph Intelligent Remediation System Core Logic
        TRIGGER[Anomaly Trigger]
        PARALLEL_PROCESSOR[Parallel Data Processor]
        REDIS_AGGREGATOR[1. Redis Data Aggregator]
        GRAFANA_DASHBOARD_GEN[2. Grafana Dashboard Generator]
        MCP_SERVER[MCP Server Model Context Protocol Server]
        LLM[LLM e.g., OpenAI API]
        K8SAPI_MCP[Kubernetes API Controlled by MCP Server]
        FEEDBACK_LOOP[Feedback Loop / Learning]

        REDIS -- 15min Data --> TRIGGER
        TRIGGER -- Anomaly Detected --> PARALLEL_PROCESSOR
        PARALLEL_PROCESSOR -- Query --> REDIS_AGGREGATOR
        PARALLEL_PROCESSOR -- Generate --> GRAFANA_DASHBOARD_GEN

        REDIS_AGGREGATOR -- Min/Max/Avg/Change --> MCP_SERVER
        GRAFANA_DASHBOARD_GEN -- Dashboard URL --> MCP_SERVER

        MCP_SERVER -- Enriched Context & Query --> LLM
        LLM -- Diagnosis & Proposed Action --> MCP_SERVER

        MCP_SERVER -- Action Request --> K8SAPI_MCP
        K8SAPI_MCP -- Execute Action --> K8SAPI

        K8SAPI_MCP -- Action Result --> MCP_SERVER
        MCP_SERVER -- Learning Data --> FEEDBACK_LOOP
    end

    subgraph Operations & Monitoring
        GRAFANA[Grafana]
        ALERT_MANAGER[Alert Manager / Notification]

        PROM --> GRAFANA
        OTEL --> GRAFANA
        MCP_SERVER -- Alert/Notification --> ALERT_MANAGER
    end

    classDef k8s fill:#f9f,stroke:#333,stroke-width:2px;
    classDef data fill:#bbf,stroke:#333,stroke-width:2px;
    classDef core fill:#ffc,stroke:#333,stroke-width:2px;
    classDef ops fill:#cfc,stroke:#333,stroke-width:2px;

    class K8SAPI,PROM,OTEL,APP,EVENT k8s
    class DATA_COLLECTOR,KAFKA,REDIS data
    class TRIGGER,PARALLEL_PROCESSOR,REDIS_AGGREGATOR,GRAFANA_DASHBOARD_GEN,MCP_SERVER,LLM,K8SAPI_MCP,FEEDBACK_LOOP core
    class GRAFANA,ALERT_MANAGER ops