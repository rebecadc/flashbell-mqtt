# Flashbell MQTT

Sistema de campainha inteligente IoT com alerta visual para pessoas com deficiência auditiva. Um visitante pressiona um botão em uma página web e o residente recebe um alerta visual (tela piscando) em outra página, tudo comunicado via protocolo MQTT.

## Arquitetura

```
┌──────────────┐      MQTT       ┌──────────────┐
│  Visitante    │── WebSocket ──▶│   Broker     │
│  (Emissor)   │                 │  Mosquitto   │
│  :8081       │                 │  :1883/:9001 │
└──────────────┘                 └──────┬───────┘
                                        │ MQTT
                                        ▼
                                 ┌──────────────┐
                                 │  Residente   │
                                 │  (Receptor)  │
                                 │  :8082       │
                                 └──────────────┘
```

## Tecnologias

- **Broker MQTT** — Eclipse Mosquitto 2
- **Clientes web** — HTML + CSS + JavaScript (vanilla)
- **Biblioteca MQTT** — [MQTT.js v4.3.7](https://cdnjs.cloudflare.com/ajax/libs/mqtt/4.3.7/mqtt.min.js) via CDN
- **Servidor web** — Nginx Alpine (Docker)
- **Orquestração** — Docker Compose

## Como executar

```bash
docker-compose up --build
```

Acesse:

- **Visitante (emissor):** http://localhost:8081
- **Residente (receptor):** http://localhost:8082

## Como funciona

1. O visitante abre `http://localhost:8081` e clica no botão "Tocar Campainha".
2. Uma mensagem JSON é publicada no tópico MQTT `casa/campainha`:
   ```json
   { "evento": "campainha_pressionada", "timestamp": 1719500000000 }
   ```
3. O residente, conectado em `http://localhost:8082`, recebe a mensagem e visualiza um alerta com a tela piscando em vermelho e branco por aproximadamente 5 segundos.
4. A latencia de ponta a ponta é calculada e exibida na tela do residente.

## Portas

| Servico          | Porta do Host | Porta do Container | Protocolo                |
| ---------------- | ------------- | ------------------ | ----------------------- |
| Broker MQTT      | 1884          | 1883               | MQTT (TCP)              |
| Broker WebSocket | 9001          | 9001               | MQTT sobre WebSockets   |
| Cliente Emissor  | 8081          | 80                 | HTTP                    |
| Cliente Receptor | 8082          | 80                 | HTTP                    |

## Estrutura do projeto

```
flashbell-mqtt/
├── docker-compose.yaml        # Orquestração dos 3 servicos
├── mosquitto.conf              # Configuração do broker Mosquitto
├── client-emissor/
│   ├── Dockerfile              # Nginx servindo a pagina do visitante
│   └── index.html              # Botão "Tocar Campainha"
└── client-receptor/
    ├── Dockerfile              # Nginx servindo a página do residente
    └── index.html              # Alerta visual com tela piscando
```

## Observações

- Não há autenticação ou TLS configurado no broker — este é um setup de desenvolvimento/demostração.
- A URL de conexão MQTT esta fixa em `ws://localhost:9001`. Para deploy remoto, altere o endereco no `index.html` de cada cliente para o hostname do broker.
