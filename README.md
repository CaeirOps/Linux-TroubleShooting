# 🐧 Laboratório Prático de Linux na AWS

> **Transformando negócios e pessoas por meio da tecnologia.**  
> _Guia prático passo a passo para quem quer aprender Linux na nuvem com segurança._

---

## 🧭 Objetivo do Laboratório

Neste laboratório, você vai aprender a:

- Criar e acessar uma instância Linux (EC2) na AWS.  
- Acessar sua máquina de forma segura via **SSH** e **Session Manager**.  
- Utilizar comandos essenciais do Linux para monitorar, investigar e resolver problemas.  
- Adotar **boas práticas de segurança** em cada etapa.

Ao final, você será capaz de realizar verificações básicas em servidores Linux, com uma visão de segurança e boas práticas de operação.

---

## ☁️ 1. Preparando o Ambiente na AWS

### 🔹 Escolhendo a Distribuição Linux

Existem diversas distribuições (distros) de Linux — como **Ubuntu**, **Debian**, **CentOS**, **Amazon Linux** e **SUSE**.  
Neste laboratório, usaremos **Amazon Linux 2023**, otimizada para rodar na AWS.

> 💡 Você pode identificar a distro de uma máquina com:  
> `cat /etc/os-release`

### 🖥️ Criando sua Instância EC2

1. Acesse o **console da AWS** e vá até **EC2 → Instâncias → Executar instância**.  
2. Dê um nome (ex: `lab-linux`).  
3. Escolha a **AMI**: _Amazon Linux 2023_.  
4. Selecione o tipo de instância: `t3.micro` (gratuito).  
5. Em **Par de chaves (login)**, escolha **Criar novo par de chaves**.  
   - Tipo de chave: **RSA**  
   - Formato: **.pem**  
   - Baixe e **guarde com segurança** — ela será usada para o SSH.
6. Configure **Rede (Network)**:
   - **VPC:** mantenha a padrão.  
   - **Subnet:** escolha uma pública.  
   - **Auto-assign Public IP:** habilite.  
7. Em **Security Group**, crie uma regra:
   - **Tipo:** SSH  
   - **Porta:** 22  
   - **Origem:** Meu IP (recomendado).  

📚 **Referências úteis:**  
- [AWS Docs – EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)  
- [AWS Docs – VPC Overview](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

---

## 🔐 2. Acessando sua Instância

### 💡 O que é SSH?

SSH (Secure Shell) é o protocolo usado para se conectar com segurança a servidores Linux.  
Ele cria um canal criptografado entre sua máquina e o servidor.

#### 🔹 Conectando via SSH:

```bash
chmod 400 chave.pem
ssh -i chave.pem ec2-user@<IP_PUBLICO>
```

- `chmod 400` protege sua chave para uso exclusivo.  
- `ec2-user` é o usuário padrão da Amazon Linux.  
- `<IP_PUBLICO>` é o IP exibido na página da instância.

> ⚠️ **Segurança:** nunca compartilhe sua chave `.pem`.  
> Ela dá acesso total à instância. Caso perca, será necessário gerar uma nova chave.

### 🧠 Recomendação: AWS Session Manager

> **É recomendado utilizar sempre o AWS Session Manager**, pois ele oferece uma forma segura e auditável de acessar o servidor diretamente pelo console ou CLI, **sem depender de chaves SSH armazenadas localmente**.  
> Todas as ações executadas podem ser registradas no CloudWatch ou S3, facilitando auditorias e rastreabilidade.  

📘 **Referência:**  
[AWS Docs – Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

---

### ⚙️ O comando `sudo` — Trabalhando com Segurança

O `sudo` (do inglês *superuser do*) permite executar comandos administrativos **sem precisar entrar como root**.  

```bash
sudo <comando>
```

Exemplo:
```bash
sudo yum update -y
```

Cada uso de `sudo` é registrado em logs como `/var/log/secure` ou `journalctl`, garantindo **rastreabilidade e segurança**.

#### 🚫 Evite o uso de `sudo su`

O comando `sudo su` abre um shell root permanente e **remove o controle de auditoria**.  
Além disso, representa um risco real de segurança:

- **Scripts maliciosos** podem aproveitar o ambiente root para **elevar privilégios** ou alterar variáveis de ambiente.  
- Você perde o controle de “quem fez o quê”.  
- Aumenta o risco de **erros acidentais**, como exclusão de arquivos do sistema.

#### ✅ Alternativas seguras

| Comando | Uso recomendado | Vantagem |
|:--|:--|:--|
| `sudo <comando>` | tarefas pontuais | mantém registro no log |
| `sudo -i` | precisa abrir um shell root temporário | cria ambiente limpo e auditável |
| `sudo su -` | alternativa com variáveis resetadas | reduz risco de herdar variáveis inseguras |

💡 **Boas práticas:**
- Use `sudo` apenas quando necessário.  
- Prefira `sudo <comando>` para ações específicas.  
- Use `sudo -i` quando for necessário permanecer no shell root.  
- Evite `sudo su` para reduzir riscos de segurança e perda de rastreabilidade.  

> 🧠 Conceito: o **root** é o “usuário superpoderoso” do Linux. Ele pode alterar, excluir e substituir qualquer arquivo do sistema.  
> Por isso, boas práticas de segurança envolvem **usar privilégios mínimos necessários**.

---

## 💻 3. Conhecendo o Sistema

Após se conectar, explore as informações básicas do ambiente:

```bash
cat /etc/os-release
uname -a
uptime
df -h
free -m
```

### 🧩 Explicações rápidas

| Comando | O que faz | Interpretação |
|:--|:--|:--|
| `cat /etc/os-release` | mostra a distribuição Linux | confirma a AMI |
| `uname -a` | exibe kernel e arquitetura | útil para suporte e compatibilidade |
| `uptime` | mostra o tempo ativo e a carga média | valores altos = sobrecarga |
| `df -h` | uso de disco | verifique se partições estão cheias |
| `free -m` | uso de memória (MB) | cache alto é normal |

> ⚠️ **Dica:** monitore sempre disco e memória — partições cheias ou falta de RAM são causas comuns de lentidão e falhas.

---

## ⚙️ 4. Monitorando Processos e Desempenho

### 🔹 `top` — visão geral do sistema

```bash
top
```

O `top` mostra processos em tempo real e consumo de CPU/memória.

- **Load Average (1,5,15 min):** indica carga do sistema.  
- **%us:** CPU usada por processos do usuário.  
- **%sy:** uso pelo sistema (kernel).  
- **%wa:** tempo aguardando I/O (disco).  

| Tecla | Função |
|:--|:--|
| `P` | ordenar por CPU |
| `M` | ordenar por memória |
| `1` | mostrar por núcleo |
| `q` | sair |

### 🎨 `htop` — interface visual

Instale e execute:

```bash
sudo yum install htop -y
htop
```

Use `F3` para buscar processos, `F9` para encerrar.  

💡 **Simule carga para observar comportamento:**

```bash
yes > /dev/null &
```

Finalize com:

```bash
sudo pkill yes
```

> ⚠️ Cuidado: comandos que rodam em loop infinito, como `yes`, devem ser finalizados — ou podem travar a instância.

---

## 🧩 5. Serviços e Tarefas Agendadas

### 🔍 Verificando serviços

```bash
sudo systemctl status nginx
sudo systemctl restart nginx
```

Se não estiver instalado:

```bash
sudo yum install nginx -y
sudo systemctl enable nginx --now
```

Verifique se o NGINX está escutando na porta 80:

```bash
sudo ss -tulnp | grep nginx
```

> Isso mostra o processo e a porta associada ao serviço.  
> 🔒 **Segurança:** apenas serviços necessários devem estar em “LISTEN”.

### ⏰ Agendando tarefas com Cron

Visualize e crie tarefas agendadas:

```bash
crontab -l
sudo crontab -l -u root
cat /etc/crontab
ls /etc/cron.*
```

Crie um cron de teste:

```bash
echo '* * * * * echo "$(date) - cron rodando" >> /tmp/cron_test.log' | sudo tee /etc/cron.d/testcron
sudo systemctl restart crond
tail -f /tmp/cron_test.log
```

> ⚠️ **Atenção:** nunca adicione scripts desconhecidos no cron — eles serão executados automaticamente com as permissões do usuário.

---

## 📜 6. Logs e Diagnóstico

### 🔹 Onde ficam os logs

```bash
cd /var/log
ls -lh
sudo tail -n 20 messages
sudo grep -i error /var/log/messages
sudo tail -f /var/log/secure
```

| Local | Descrição |
|:--|:--|
| `/var/log/messages` | eventos do sistema |
| `/var/log/secure` | autenticação e uso do sudo |
| `/var/log/nginx/error.log` | erros do NGINX |
| `/var/log/app.log` | logs customizados |

> 💡 Monitore sempre logs de segurança (`secure`, `auth.log`) — eles revelam tentativas de acesso e falhas de permissão.

---

## 🌐 7. Rede e Conectividade

### 🔹 Testes básicos

```bash
ping 8.8.8.8
curl -v http://localhost
nslookup google.com
traceroute 8.8.8.8
```

### 🔹 Verificando conexões e portas

```bash
sudo ss -tulnp
sudo ss -tn state established
```

> 🔍 O comando `ss` (ou `netstat`) mostra portas abertas e conexões ativas.  
> ⚠️ **Segurança:** mantenha apenas as portas realmente necessárias abertas no Security Group e firewall interno.

---

## ✅ 8. Encerramento do Laboratório

### 🧾 Checklist final

| Item | Validado |
|:--|:--|
| EC2 criada e acessível | ✅ |
| Acesso seguro via SSH ou Session Manager | ✅ |
| Serviço ativo e auditado | ✅ |
| Logs verificados | ✅ |
| Rede validada | ✅ |

### 🧹 Limpeza

```bash
sudo pkill yes
sudo rm -f /tmp/cron_test.log
```

> Lembre-se de encerrar a instância EC2 ao final do laboratório para evitar custos.

---

## 🧰 Recursos Extras

- [AWS Docs – Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)  
- [AWS Docs – EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)  
- [AWS Docs – Amazon Linux 2023 User Guide](https://docs.aws.amazon.com/linux/al2023/ug/what-is-amazon-linux.html)

---

## 🧡 Conclusão

Parabéns! 🎉  
Você concluiu um laboratório completo de Linux na AWS, aplicando conceitos práticos e de segurança.  

Agora você entende como:  
- Criar e acessar instâncias com segurança.  
- Usar comandos essenciais do Linux.  
- Ler logs, monitorar serviços e validar rede.  
- Adotar boas práticas que protegem seu ambiente e dados.  

> “A curiosidade é a melhor ferramenta de aprendizado — explore, teste e aprenda com segurança.”

---
