# Atividade aula 3 Alaelson

# Relatório de Prática: Configuração de Redes em Máquinas Virtuais (NAT vs. Bridge)

## 1. Identificação
* **Nome Completo:** Max Clay da Silva
* **Turma:** 913-A
* **Data:** 25/10/2026

---

## 1.5 Links: 

- Link das Imagens: https://github.com/mcs77-cyber/lsor-2026.2-sysdev-atividades-Max-Clay-da-Silva/tree/main/imagens/imagens%20aula%203

## 2. Objetivo
Analisar, configurar e comparar o comportamento de interfaces de rede em máquinas virtuais utilizando os modos **NAT (Network Address Translation)** e **Bridge (Ponte)**, validando endereçamento IP, máscara de sub-rede, gateway padrão, tabela de rotas e a acessibilidade a partir da máquina física (host).

---

## 3. Ambiente
* **Hipervisor / Sistema:** [VirtualBox / Linux Ubuntu Server]
* **Modos de Rede Testados:** 
  * NAT (Network Address Translation)
  * Placa em Modo Bridge (Ponte)
* **Endereçamento Padrão de Referência:**
  * **Modo NAT:** Sub-rede interna padrão 
  * **Modo Bridge:** Mesma sub-rede da rede física local 

---

## 4. Procedimento
Comandos executados:

1. #### Exibir interfaces de rede, endereços IP e MAC:

   `ip addr`

2. #### Exibir a tabela de roteamento e o gateway padrão:

   `ip route`

3. #### Testar conectividade local (Loopback):

   `ping -c 4 127.0.0.1`

4. #### Testar conectividade com o Gateway:

   `ping -c 4 10.0.2.2`

5. #### Testar conectividade com a Internet (DNS/Externa):

   `ping -c 4 8.8.8.8`

## 5. Testes

## 5.1 Comandos:

#### A. comando `ip addr`:

$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65535 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/127 scope host 
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a3:b4:c1 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 86399sec preferred_lft 86399sec
    inet6 fe80::a00:27ff:fea3:b4c1/64 scope link

### Análise: Identifica-se a interface eth0, o endereço MAC (08:00:27:a3:b4:c1), o endereço IP (10.0.2.15) e a máscara de sub-rede (/24, equivalente a 255.255.255.0).

#### B. comando `ip route`:

$ ip route
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100

### Análise: O Gateway Padrão está configurado em 10.0.2.2 (endereço reservado do NAT do hipervisor), por onde todo o tráfego externo é encaminhado.

#### C. `ping`:

#### C.1 Ping para o Loopback (Camada de Rede Interna / Software):

$ ping -c 4 127.0.0.1
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.041 ms

#### C.2 Ping para o Gateway (Camada 3 - Roteamento Local):

$ ping -c 4 10.0.2.2
64 bytes from 10.0.2.2: icmp_seq=1 ttl=64 time=0.825 ms

#### C.3 Ping para a Internet / DNS Externo (Conectividade Externa via NAT):

$ ping -c 4 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=14.2 ms

## 5.2. Tabela Comparativa: NAT x Bridge:

| Critério de Comparação | Modo NAT (Network Address Translation) | Modo Bridge (Ponte)|
| :---: | :---: | :---: |
|Endereço IP da VM | Privado, gerado pelo DHCP interno do hipervisor | Da mesma faixa da rede física local, fornecido pelo roteador da casa/escritório |
| Gateway Padrão | Endereço interno do hipervisor | O roteador físico real da rede local |
| Host acessa a VM? | Não, pois o host está fora da sub-rede privada NAT e o tráfego de fora para dentro é bloqueado (salvo com regras específicas de redirecionamento de porta) | Sim, pois a VM passa a ser um dispositivo independente visível na mesma rede física do host.

## 6. Problemas e Soluções

  Problema: No modo NAT, ao tentar realizar um teste de ping a partir da máquina física (host) em direção à máquina virtual (10.0.2.15), a comunicação falha e o pacote é perdido.

  ### 6.1 Diagnóstico:

   Por que isso ocorre? O IP 10.0.2.15 pertence a uma rede privada virtual criada pelo software de virtualização. O host físico não possui rota para essa sub-rede interna. O NAT opera mascarando os pacotes que saem da VM para a internet, mas não permite conexões iniciadas de fora (do host para a VM) sem regras explícitas de Port Forwarding (Redirecionamento de Portas).

   #### 6.2 Solução aplicada: 
   
   Para permitir que o host acesse diretamente a VM, alterou-se o adaptador de rede do hipervisor de NAT para Modo Bridge, fazendo com que a VM recebesse um IP diretamente do roteador físico da rede local.

## 7. Conclusão

A prática permitiu validar o funcionamento prático das camadas de rede (Camada 2 com endereços MAC e Camada 3 com endereços IP e rotas). Comprovou-se que o modo NAT é ideal para prover acesso à internet para a VM sem consumir IPs da rede física, enquanto o modo Bridge é essencial quando a máquina virtual precisa atuar como um servidor acessível por outros dispositivos da mesma rede local. O diagnóstico de falhas com o comando ping demonstrou a importância de analisar a tabela de roteamento e o isolamento de sub-redes.
