# Instalação e Configuração do Loki

Este guia descreve os passos para instalar e configurar o Loki em um sistema RHEL/CentOS. Algumas etapas, como configuração de proxy e desabilitação de firewall, são opcionais e dependem do ambiente de rede e segurança.

## ✨ Requisitos
- Acesso root ou sudo
- Sistema com `dnf` (ex: CentOS 8 ou RHEL 8)

---

## 🔄 (Opcional) Configurar Proxy de Rede
Se o ambiente exigir uso de proxy para acesso à internet:
```bash
export {http,https,ftp}_proxy="http://SEU_PROXY_AQUI:8080"
export NO_PROXY="localhost,127.0.0.1,REDE_LOCAL"
```

---

## 📦 1. Atualizar o Sistema e Instalar Ferramentas
```bash
dnf update -y
dnf install -y wget epel-release htop vim
```

---

## 📁 2. Importar a Chave GPG do Grafana
```bash
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
rpm --import gpg.key
```

---

## 📂 3. Instalar o Loki
```bash
dnf install loki -y
```
> ⚠️ Caso o comando acima não funcione, é necessário adicionar o repositório do Grafana manualmente.

---

## 📄 4. Configurar o Loki

### Fazer backup do arquivo original
```bash
mv /etc/loki/config.yml /etc/loki/config_original.yml
```

### Editar novo arquivo de configuração
```bash
vim /etc/loki/config.yml
```

### Definir permissões
```bash
chmod 755 /etc/loki/config.yml
```

---

## 🔥 (Opcional) Desabilitar o Firewall
Somente em ambientes de teste ou onde o acesso externo esteja controlado:
```bash
systemctl stop firewalld
systemctl disable firewalld
```

---

## ▶️ 5. Iniciar e Habilitar o Loki
```bash
systemctl daemon-reload
systemctl start loki
systemctl enable loki
systemctl status loki
```

---

## 🌐 6. Acesso via Navegador
Acesse o Loki via navegador:
```
http://<SEU-ENDERECO-LOKI>:3100
```
> Substitua `<SEU-ENDERECO-LOKI>` pelo endereço IP ou hostname do seu servidor.

---

## 🛠️ 7. Verificação de Logs
Em caso de problemas no start do Loki:
```bash
journalctl -u loki -f
vim /var/log/messages
```

---

## 🔹 8. Integração com Grafana
1. Acesse o painel do Grafana
2. Navegue até `Configuration > Data Sources`
3. Adicione uma nova fonte de dados do tipo **Loki**
4. Configure a URL como:
```
http://<SEU-ENDERECO-LOKI>:3100
```

---

## 🚀 Concluído!
O Loki está instalado e funcionando, pronto para ser integrado ao Grafana e visualizar logs com performance e praticidade.

---
