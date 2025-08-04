# loki-install
Instalação do Grafana Loki no CentOS/RHEL
Este guia descreve como preparar um servidor Linux (CentOS, RHEL, AlmaLinux etc.) para instalar e configurar o Loki, o agregador de logs da Grafana Labs.

✅ Requisitos
Distribuição baseada em RHEL (8+)

Acesso root (ou via sudo)

Acesso à internet (direto ou via proxy opcional)
🔧 1. [Opcional] Configurar proxy de rede
Se estiver atrás de proxy, exporte as variáveis de ambiente:

bash
Copiar
Editar
export {http,https,ftp}_proxy="http://seu.proxy:porta"
export NO_PROXY="localhost,127.0.0.1"
💡 Para tornar isso permanente, salve no arquivo:
/etc/profile.d/proxy.sh

🔄 2. Atualizar o sistema
bash
Copiar
Editar
dnf update -y
reboot
🧰 3. Instalar ferramentas essenciais
bash
Copiar
Editar
dnf install -y open-vm-tools wget epel-release htop vim
🔐 4. Importar chave GPG da Grafana
bash
Copiar
Editar
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
rpm --import gpg.key
📦 5. Instalar o Loki
O pacote loki pode não estar disponível diretamente via dnf. Neste caso, use o binário oficial:

bash
Copiar
Editar
wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
chmod +x loki-linux-amd64
mv loki-linux-amd64 /usr/local/bin/loki
⚙️ 6. Configurar o Loki
Crie a pasta de configuração:

bash
Copiar
Editar
mkdir -p /etc/loki
Exemplo básico de configuração em /etc/loki/config.yml:

yaml
Copiar
Editar
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9095

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  max_chunk_age: 1h
  chunk_target_size: 1048576
  chunk_retain_period: 30s
  wal:
    enabled: true
    dir: /tmp/wal

schema_config:
  configs:
    - from: 2020-10-15
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/index
    cache_location: /tmp/loki/boltdb-cache
    shared_store: filesystem
  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
Salve o arquivo como /etc/loki/config.yml.

🛠️ 7. Criar serviço systemd para Loki
Crie o arquivo: /etc/systemd/system/loki.service

ini
Copiar
Editar
[Unit]
Description=Loki Log Aggregator
After=network.target

[Service]
ExecStart=/usr/local/bin/loki -config.file=/etc/loki/config.yml
Restart=on-failure

[Install]
WantedBy=multi-user.target
▶️ 8. Iniciar e habilitar o serviço
bash
Copiar
Editar
systemctl daemon-reload
systemctl start loki
systemctl enable loki
systemctl status loki
🔥 [Opcional] Desabilitar o Firewall
Use com cuidado. Apenas se o ambiente permitir.

bash
Copiar
Editar
systemctl stop firewalld
systemctl disable firewalld
📎 9. Testar Loki com Grafana
No Grafana:

Vá para Admin → Data Sources

Adicione nova fonte de dados

Escolha Loki

Em URL: http://<ip_do_servidor>:3100

Salve e teste

📁 Estrutura esperada
swift
Copiar
Editar
/usr/local/bin/loki
/etc/loki/config.yml
/etc/systemd/system/loki.service
📌 Dica final
Para monitorar logs:

bash
Copiar
Editar
journalctl -u loki -f
