# Relatório de Prática — Acesso Remoto Seguro com OpenSSH e Redirecionamento de Portas no VirtualBox

## 1. Identificação

* **Nome completo:** Max Clay da Silva
* **Turma:** 913-A
* **Data:** 08/11/2026
* **Título da prática:** Acesso Remoto Seguro com OpenSSH e Redirecionamento de Portas no VirtualBox

---

## 2. Objetivo

O objetivo desta prática foi configurar, diagnosticar e validar o acesso remoto seguro via **OpenSSH** a uma máquina virtual isolada em rede **NAT** no VirtualBox. Ao longo da atividade, foram exercitados:
* Instalação de utilitários clássicos de diagnóstico de rede (`net-tools` e `traceroute`) no Ubuntu Server 26.04 LTS.
* Inspeção de interfaces de rede (`ifconfig` e interface de loopback `lo`), análise da tabela de roteamento (`route -n`) e rastreamento de pacotes (`traceroute`).
* Verificação e gerenciamento do daemon `ssh` (`openssh-server`) via `systemd`.
* Configuração de redirecionamento de portas (*Port Forwarding*) no VirtualBox para expor a porta `22` da VM na porta `5222` do host Windows.
* Monitoramento dos estados de conexão TCP (`LISTENING`, `ESTABLISHED` e `TIME_WAIT`) com `netstat -an` antes, durante e depois da sessão SSH, além da auditoria de sessões ativas no Linux com o comando `w`.

---

## 3. Ambiente

* **Host (Máquina Física):** Sistema Operacional Windows com cliente OpenSSH e utilitários `ipconfig` / `netstat`.
* **Hipervisor:** Oracle VM VirtualBox.
* **Máquina Virtual (Guest):** `ubuntu-server` — Ubuntu Server 26.04 LTS (768 MB de RAM, 1 vCPU).
* **Configuração de Rede:**
  * **Modo do Adaptador 1:** NAT (*Network Address Translation*).
  * **IP interno da VM (Guest):** `10.0.2.15` (interface `enp0s3`) e `127.0.0.1` (interface de loopback `lo`).
  * **Gateway padrão NAT:** `10.0.2.2`.
  * **Regra de Redirecionamento de Portas:** Protocolo `TCP` | Host `127.0.0.1:5222` $\rightarrow$ Guest `10.0.2.15:22`.

---

## 4. Procedimento

### Passo 1 — Instalação de ferramentas de rede e inspeção das interfaces (`ifconfig` e `lo`)

Nesta etapa inicial, atualizei os repositórios e instalei os pacotes `net-tools` (que fornece os comandos `ifconfig`, `route` e `netstat`) e `traceroute`. Em seguida, executei `ifconfig` para identificar o endereço IP atribuído pelo DHCP interno do VirtualBox à interface principal da VM e inspecionei especificamente a interface de *loopback* (`lo`).

```bash
sudo apt update
sudo apt install -y net-tools traceroute
ifconfig
ifconfig lo
```

![Inspeção das interfaces de rede no Linux com ifconfig](imagens/Imagens%20aula%204/aula4_descricao_linux-ifconfig.png)

![Detalhes da interface de loopback lo](imagens/Imagens%20aula%204/aula4_descricao_linux-lo.png)

A saída de `ifconfig` confirma que a interface de rede principal recebeu o endereço IP `10.0.2.15` (máscara `255.255.255.0`), típico do modo NAT do VirtualBox, enquanto `ifconfig lo` mostra a interface local `127.0.0.1` ativa (`UP LOOPBACK RUNNING`) para comunicação interna do sistema.

---

### Passo 2 — Diagnóstico da tabela de rotas (`route -n`) e rastreamento de saltos (`traceroute`)

Para compreender como o tráfego sai da máquina virtual em direção ao host e à internet, consultei a tabela de roteamento do kernel sem resolução de nomes (`route -n`) e executei um rastreamento de rota (`traceroute`) para observar os saltos de rede.

```bash
route -n
traceroute 8.8.8.8
```

![Tabela de roteamento do kernel com route -n](imagens/Imagens%20aula%204/aula4_descricao_route_-n.png)

![Rastreamento de rota com traceroute](imagens/Imagens%20aula%204/aula4_descricao_traceroute.png)

O comando `route -n` exibe a rota padrão (`0.0.0.0`) apontando para o gateway `10.0.2.2` (roteador virtual do VirtualBox), e o `traceroute` demonstra o primeiro salto passando exatamente por esse gateway antes de alcançar a rede externa.

---

### Passo 3 — Verificação do serviço OpenSSH Server (`systemctl status ssh`)

Antes de configurar o acesso externo, verifiquei se o servidor SSH estava instalado, ativo e escutando conexões na porta padrão `22` dentro do Ubuntu Server.

```bash
sudo systemctl status ssh
```

![Status do serviço OpenSSH Server no Ubuntu Server](imagens/Imagens%20aula%204/aula4_descricao_system-status.png)

A saída mostra o serviço `ssh.service` com o estado `active (running)` e habilitado para iniciar junto com o sistema, confirmando que a VM está pronta para receber conexões remotas.

---

### Passo 4 — Configuração do Redirecionamento de Portas (NAT) no VirtualBox

Como o modo NAT isola a VM da rede externa impedindo conexões diretas iniciadas pelo host, acessei as configurações da VM no VirtualBox (**Configurações $\rightarrow$ Rede $\rightarrow$ Adaptador 1 $\rightarrow$ Avançado $\rightarrow$ Redirecionamento de Portas**) e cadastrei a regra TCP mapeando a porta `5222` do loopback do host Windows (`127.0.0.1`) para a porta `22` da VM (`10.0.2.15`).

![Regra de redirecionamento de portas NAT configurada no VirtualBox](imagens/Imagens%20aula%204/aula4_descricao_network-rule.png)

Com a regra salva, o VirtualBox passa a escutar requisições locais na porta `5222` do Windows e encaminhá-las de forma transparente para o servidor OpenSSH da VM.

---

## 5. Testes, Evidências e Tabela de Comandos

#### Links: https://github.com/mcs77-cyber/lsor-2026.2-sysdev-atividades-Max-Clay-da-Silva/tree/main/imagens/Imagens%20aula%204

### Teste A — Estado da porta `5222` no Host *antes* da conexão SSH (OBS: testes feitos no ambiente linux, mas mostrando o comando do windows para mais fácil entendimento)

No terminal do host, verifiquei a tabela de conexões e portas ativas filtrando pela porta `5222` antes de abrir qualquer cliente SSH, para confirmar que o VirtualBox já havia reservado o socket de escuta.

```powershell
ipconfig
netstat -an | findstr 5222
```

![Estado da porta 5222 no Windows antes da conexão](imagens/Imagens%20aula%204/aula4_descricao_windows_antes.png)

A saída não exibe nenhuma conexão,mostrando como até agora nada foi feito.
---

### Teste B — Conexão remota SSH e monitoramento *durante* a sessão ativa

Em seguida, iniciei o acesso remoto a partir do host direcionando o cliente SSH para `127.0.0.1` na porta `5222`. Já autenticado dentro da VM, executei o comando `w` para auditar as sessões ativas e, simultaneamente, consultei novamente o `netstat` no Windows para observar a mudança de estado da conexão TCP.

```bash
# No terminal do Host (iniciando a conexão):
ssh -p 5222 administrador@127.0.0.1

# Dentro da sessão SSH no Ubuntu Server:
w
```

```powershell
# Em outro terminal no Host (durante a sessão ativa):
netstat -an | findstr 5222
```

![Conexão remota estabelecida via SSH na porta 5222](imagens/Imagens%20aula%204/aula4_descricao_sessao-ssh_conexao.png)

![Auditoria da sessão remota ativa no Linux com o comando w](imagens/Imagens%20aula%204/aula4_descricao_sessao-ssh_comando_w.png)

![Estado da porta 5222 no Windows durante a conexão SSH](imagens/Imagens%20aula%204/aula4_descricao_windows_durante.png)

Os prints comprovam o funcionamento completo do túnel: o prompt mudou para `administrador@ubuntu-server`, o comando `w` listou a sessão `pts/` conectada a partir do gateway `10.0.2.2` e o `netstat` passou a exibir o par de sockets no estado `ESTABLISHED`.

---

### Teste C — Encerramento da sessão e verificação *depois* da conexão

Por fim, encerrei a sessão remota digitando `exit` no terminal SSH e executei uma última verificação da porta `5222` no host para confirmar o encerramento limpo do canal TCP.

```bash
# Encerrando a sessão SSH:
exit
```

```powershell
# No Host após encerrar a sessão:
netstat -an | findstr 5222
```

![Estado da porta 5222 no Windows depois de encerrar a conexão](imagens/Imagens%20aula%204/aula4_descricao_windows_depois.png)

Após o logout, a conexão ativa é encerrada (passando brevemente por `TIME_WAIT` até ser liberada), restando apenas o socket `127.0.0.1:5222` em `LISTENING`, pronto para novas conexões.

---

### Tabela de Comandos Utilizados

| Comando | O que significa e/ou o que faz |
| :--- | :--- |
| `sudo apt install -y net-tools traceroute` | Instala os pacotes de utilitários de diagnóstico de rede no Ubuntu (`ifconfig`, `route`, `netstat` e `traceroute`). |
| `ifconfig` / `ifconfig lo` | Exibe as configurações das interfaces de rede ativas (IP, máscara, MAC e estatísticas de pacotes) ou detalha especificamente a interface de *loopback* (`127.0.0.1`). |
| `route -n` | Mostra a tabela de roteamento IP do kernel em formato numérico (sem tentar resolver nomes DNS), identificando o gateway padrão (`10.0.2.2`). |
| `traceroute <destino>` | Mapeia o caminho percorrido pelos pacotes até um destino, exibindo cada roteador (salto) intermediário e o tempo de resposta. |
| `sudo systemctl status ssh` | Consulta o estado atual do serviço OpenSSH Server no `systemd` (verificando se está ativo e em execução). |
| `sudo systemctl enable --now ssh` | Habilita o serviço SSH para iniciar automaticamente no boot (`enable`) e o inicia imediatamente (`--now`). |
| `ipconfig` | Utilitário do Windows que exibe a configuração IP dos adaptadores de rede do computador host. |
| `netstat -an \| findstr 5222` | Lista todas as conexões e portas TCP/UDP abertas em formato numérico (`-an`) e filtra apenas as linhas referentes à porta `5222` (`LISTENING`, `ESTABLISHED`, `TIME_WAIT`). |
| `ssh -p 5222 administrador@127.0.0.1` | Inicia uma conexão remota criptografada via SSH com o usuário `administrador` no endereço local `127.0.0.1`, especificando a porta customizada `5222` (`-p`). |
| `w` | Exibe quais usuários estão logados no servidor Linux no momento, de qual IP/terminal vieram, há quanto tempo estão conectados e qual processo estão executando. |

---

## 6. Problemas e Soluções

* **Sintoma:** Ao tentar conectar pelo host utilizando `ssh -p 5222 administrador@127.0.0.1`, o terminal retorna a mensagem de conexão recusada:
  ```text
  ssh: connect to host 127.0.0.1 port 5222: Connection refused
  ```
* **Causa principal:** A regra de redirecionamento de portas (*Port Forwarding*) não foi cadastrada ou está incorreta nas configurações de rede NAT da máquina virtual no VirtualBox, ou o serviço `ssh` (`openssh-server`) está parado/inativo dentro do Linux Guest.
* **Correção aplicada:**
  1. **Verificar o serviço SSH na VM:**
     ```bash
     sudo systemctl status ssh
     ```
     Caso esteja parado ou inativo, ativá-lo e iniciá-lo imediatamente:
     ```bash
     sudo systemctl enable --now ssh
     ```
  2. **Conferir o Redirecionamento de Portas no VirtualBox:** acessar **Configurações $\rightarrow$ Rede $\rightarrow$ Adaptador 1 $\rightarrow$ Avançado $\rightarrow$ Redirecionamento de Portas** e garantir que a regra esteja configurada com os parâmetros exatos:
     * **Protocolo:** `TCP`
     * **IP do Host:** `127.0.0.1`
     * **Porta do Host:** `5222`
     * **IP do Convidado (Guest):** `10.0.2.15` *(ou em branco)*
     * **Porta do Convidado:** `22`
  3. **Validar a abertura da porta no host:**
     ```powershell
     netstat -an | findstr 5222
     ```
     Confirmando a presença do estado `LISTENING` antes de repetir o comando `ssh`.

---

## 7. Conclusão

Compreender o fluxo de portas e o mapeamento NAT entre o host e a máquina virtual é essencial para a administração de sistemas e para a eficiência no desenvolvimento de software. Esse domínio transforma o diagnóstico de falhas de rede de um processo empírico de tentativa e erro em uma análise técnica estruturada — permitindo isolar rapidamente se um bloqueio ocorre no serviço interno do servidor (`systemctl`), na tabela de rotas (`route -n`) ou na camada de tradução de portas do hipervisor (`netstat`) —, garantindo conectividade, rastreabilidade e segurança em arquiteturas virtualizadas.
