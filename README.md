# N8n-scalavel-Docker#
<img width="1408" height="768" alt="N8n-estrutura-d-scala" src="https://github.com/user-attachments/assets/a5a4a443-c85e-4c89-a534-968b6ad6042f" />


# Documentação de Implantação: n8n (Queue Mode)
Não esqueça de renomear e tirar o -main ou -worker de docker-compose.yml é apenas para identificar os arquivos.

## 1. Visão Geral
Esta documentação descreve a configuração do **n8n** operando em modo de fila (*Queue Mode*), utilizando **PostgreSQL** para persistência e **Redis** como broker de mensagens. Esta arquitetura permite a separação entre o plano de controle (*n8n-main*) e o plano de processamento (*n8n-worker*), garantindo escalabilidade e alta disponibilidade.

---

## 2. Diagrama da Arquitetura
A imagem abaixo ilustra como o tráfego flui entre o Proxy, o Main (orquestrador) e os Workers (executores).



* **Main (Control Plane):** Recebe requisições via webhook e gerencia a fila no Redis.
* **Redis:** Armazena as tarefas na fila (Job Queue).
* **Worker:** Consome as tarefas da fila, executa os fluxos e reporta o sucesso ao Postgres.

---

## 3. Estrutura de Infraestrutura (Hardware: 4 vCPU / 8GB RAM)
Com este hardware, o balanceamento de carga é essencial.

| Componente | Função | Recurso Alocado |
| :--- | :--- | :--- |
| **n8n-main** | Interface/API | 1 vCPU / 2GB RAM |
| **n8n-worker** | Processamento | 3 vCPU / 4GB RAM |
| **Postgres/Redis** | Banco/Cache | 0.5 vCPU / 2GB RAM |

---

## 4. Arquitetura de Comunicação (Rede)
Abaixo, o modelo de como as portas devem ser configuradas para comunicação externa:



1.  **Main:** Escuta na porta `5678`.
2.  **Postgres:** Porta `5454` (Externa) mapeada para `5432`.
3.  **Redis:** Porta `6379` (Externa) para conexão dos workers.

---

## 5. Procedimento de Implantação (Checklist)

### A. Preparação do Ambiente
1. **Firewall:** O passo mais importante.
   ```bash
   # Exemplo (ajuste conforme seu firewall)
   ufw allow from <IP_DO_WORKER> to any port 5454 proto tcp
   ufw allow from <IP_DO_WORKER> to any port 6379 proto tcp
