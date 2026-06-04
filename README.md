# Como Conectar o Open WebUI ao Ollama Local no Linux (Com Suporte a GPU) IA Funcionando totalmente offline no seu computador!

> 💻 **Ambiente de Execução:** Este tutorial foi testado e homologado em uma máquina rodando **Ubuntu 26.04 LTS** com uma placa de vídeo **NVIDIA GeForce RTX 5050**.

Este guia prático foi criado para resolver os principais problemas de integração entre o **Open WebUI** (rodando no Docker) e o **Ollama** (rodando nativamente no Linux), garantindo a comunicação correta entre eles e o uso total da sua placa de vídeo (GPU).

---
<img width="939" height="663" alt="image" src="https://github.com/user-attachments/assets/97e03425-ee57-492b-b9b4-f34805583bd8" />
<img width="1624" height="1305" alt="image" src="https://github.com/user-attachments/assets/d067c124-ee6e-4d0a-bd73-2bb8361d668a" />


---

## 🚀 Cenário Encontrado e Diagnóstico

Ao tentar conectar a interface ao Ollama, dois erros comuns de infraestrutura do Docker costumam acontecer:

1. **"Open WebUI Backend Required":** Ocorre quando há um descompasso no cache do Docker ou o uso de imagens antigas/incompletas (apenas frontend). A solução é forçar o `pull` da imagem correta `:main` que integra o servidor backend em Python com o frontend.
2. **"Failed to fetch models" / "No models available":** Ocorre porque o Docker roda em uma rede virtualizada isolada. Por padrão, o Ollama do Linux recusa conexões que não venham estritamente de `localhost` (127.0.0.1 do host), bloqueando o container. Além disso, rotas como `host.docker.internal` exigem configurações extras no Linux.

Abaixo está o passo a passo definitivo para corrigir todos esses pontos e rodar o modelo **qwen2.5:8b** com aceleração por hardware.

---

## 🛠️ Passo 1: Configurar o Ollama (Host) para Aceitar Conexões 

Aqui eu parto do pressuposto de que você já tem o Ollama instalado e já carregado com algum modelo de LLM! Após isso, Precisamos dizer ao serviço do Ollama no Linux que ele pode responder a requisições de outras interfaces de rede (como a ponte do Docker).

1. No terminal do Linux, abra o editor de configuração do serviço:
```bash
   sudo systemctl edit ollama.service
2 - Adicione as linhas abaixo exatamente abaixo de [Service]:
[Service]
   Environment="OLLAMA_HOST=0.0.0.0"

3 - Salve o arquivo, feche o editor e reinicie o gerenciador de serviços para aplicar:

sudo systemctl daemon-reload
sudo systemctl restart ollama

🐳 Passo 2: Criar o Arquivo docker-compose.yml Corrigido
Para evitar problemas complexos de roteamento de IP e pontes virtuais do Docker no Linux, a melhor estratégia é usar o network_mode: "host". Isso remove o isolamento de rede do container, fazendo com que o Open WebUI acesse o Ollama diretamente através do IP local padrão.

Crie um arquivo chamado docker-compose.yml em sua pasta de preferência:

version: '3.8'

services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    network_mode: "host"  # Compartilha a rede diretamente com o Linux host
    volumes:
      - open-webui:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=[http://127.0.0.1:11434](http://127.0.0.1:11434)
      - PORT=3000          # Define a porta física onde o Open WebUI vai rodar
    restart: always

volumes:
  open-webui:

⚡ Passo 3: Limpar o Cache e Iniciar o Container
Para garantir que nenhuma configuração antiga ou imagem corrompida interfira, rode a sequência abaixo no terminal (dentro da pasta do arquivo criado):

# 1. Derruba qualquer instância anterior
docker compose down

# 2. Garante o download da imagem oficial completa e atualizada
docker compose pull

# 3. Inicializa o serviço em segundo plano (detached mode)
docker compose up -d

⚙️ Passo 4: Sincronizar os Modelos na Interface
Se ao entrar no painel você ainda visualizar a mensagem de que não há modelos:

1 - Clique no botão "Manage Connections" (ou vá em Settings > Connections).

2 - Certifique-se de que o campo Ollama API URL está preenchido com http://127.0.0.1:11434.

3 - Clique no botão de atualizar/sincronizar (ícone de seta circular).

O seu modelo qwen2.5:8b (ou qualquer outro baixado via terminal) aparecerá instantaneamente na lista de seleção superior.





