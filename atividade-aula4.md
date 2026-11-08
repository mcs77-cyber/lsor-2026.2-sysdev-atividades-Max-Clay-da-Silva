# Acesso Remoto Seguro com OpenSSH e Redirecionamento de Portas no VirtualBox
### Nome: Max Clay da Silva 
### Turma: 913-A
### Data: 08/11/2026

## Objetivo: diagnóstico de rotas, controle de portas com netstat e acesso remoto SSH em ambiente NAT.
## Ambiente: configurações do host Windows e da VM Ubuntu Server 26.04 LTS no VirtualBox (modo de rede, IPs, porta redirecionada).
## Procedimento: instalação de net-tools/traceroute, verificação do openssh-server, uso de route -n, w, ipconfig, netstat -an e configuração do NAT, em ordem.
## Problemas e Soluções: 

#### Sintoma: O comando ssh -p 5222 administrador@127.0.0.1 retorna uma mensagem informando que a conexão foi recusada (ssh: connect to host 127.0.0.1 port 5222: Connection refused).

    Causa principal: A regra de redirecionamento de portas (Port Forwarding) não foi configurada corretamente nas configurações de rede da máquina virtual no VirtualBox, ou o serviço ssh (OpenSSH Server) não está ativo e rodando dentro do Linux Guest.

#### Correção

  - Temos que verificar se o serviço SSH está rodando na VM:

    `sudo systemctl status ssh`

  - Se estiver parado ou inativo:

     `sudo systemctl enable --now ssh`

  depois, verificamos o Redirecionamento de Portas no VirtualBox:

  Com a VM desligada, vamos em configurações, depois em: Rede > Adaptador 1 > Avançado > Redirecionamento de Portas.

  depois, é necessário certificar se a regra está cadastrada com estes parâmetros:

        Protocolo: TCP

        IP do Host: 127.0.0.1

        Porta do Host: 5222

        IP do Convidado (Guest): (pode deixar em branco ou colocar o IP da VM 10.0.2.15)

        Porta do Convidado: 22

  por fim, verificamos a porta no host:

        `netstat -an | grep 522` (ou netstat -an | findstr 5222 no windows)

### Conclusão

Compreender o fluxo de portas, especialmente o mapeamento NAT entre o host e o guest, é necessário para administração de sistemas e para compreenção e enficiencia maxima dos desenvolvedores, Já que esse entendimento transforma a solução de problemas de um processo de tentativa e erro em uma análise técnica estruturada, garantindo a segurança, a conectividade e a eficiência em arquiteturas virtualizadas.
