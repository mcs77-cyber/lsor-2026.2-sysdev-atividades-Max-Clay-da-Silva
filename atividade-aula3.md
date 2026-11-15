# Relatório de Prática — Configuração de Redes em Máquinas Virtuais (NAT vs. Bridge)

## 1. Identificação

* **Nome completo:** Max Clay da Silva
* **Turma:** 913-A
* **Data:** 25/10/2026
* **Título da prática:** Configuração de Redes em Máquinas Virtuais (NAT vs. Bridge)

---

## 2. Objetivo

O objetivo desta aula prática foi analisar, configurar e comparar o comportamento das interfaces de rede em máquinas virtuais operando nos modos **NAT (*Network Address Translation*)** e **Bridge (Ponte)**. Ao longo da atividade, foram validados:
* Identificação de interfaces de rede, endereços físicos (MAC), endereços lógicos (IPv4/IPv6) e máscaras de sub-rede com `ip addr`.
* Análise da tabela de roteamento do sistema e identificação do gateway padrão com `ip route`.
* Diagnóstico de conectividade em camadas (pilha TCP/IP interna, gateway virtual e internet externa) com o comando `ping`.
* Comparação prática de alcance e isolamento de rede entre os modos **NAT** e **Bridge** a partir da máquina física (host).

---

## 3. Ambiente

* **Plataforma de virtualização:** Oracle VM VirtualBox
* **Máquina Virtual:** `ubuntu-server` — Ubuntu Server 26.04 LTS (768 MB de RAM, 1 vCPU)
* **Modos de rede testados:**
  * **Modo NAT (*Network Address Translation*):** utiliza uma sub-rede privada virtual (`10.0.2.0/24`) gerenciada pelo DHCP interno do VirtualBox, com gateway padrão em `10.0.2.2`.
  * **Modo Bridge (Ponte):** conecta a placa virtual diretamente à rede física local, permitindo que a VM receba um endereço IP da mesma sub-rede do roteador físico.

---

## 4. Procedimento

### Passo 1 — Inspeção das interfaces de rede, endereços IP e MAC (`ip addr`)

Nesta primeira etapa, com a máquina virtual configurada em modo NAT, executei o comando `ip addr` para listar todas as interfaces de rede disponíveis no Ubuntu Server e identificar seus respectivos endereços MAC, endereços IPv4 e máscaras de sub-rede.

```bash
ip addr
```

![Inspeção das interfaces de rede com ip addr](imagens/imagens%20aula%203/aula_3_IP_addr.png)

A saída exibe a interface de *loopback* (`lo`) com o endereço `127.0.0.1/8` e a interface de rede principal (`eth0` / `enp0s3`) ativa (`state UP`), possuindo endereço MAC `08:00:27:a3:b4:c1`, endereço IPv4 `10.0.2.15` e máscara de sub-rede `/24` (`255.255.255.0`), atribuídos dinamicamente via DHCP interno do VirtualBox.

---

### Passo 2 — Exibição da tabela de roteamento e gateway padrão (`ip route`)

Em seguida, consultei a tabela de roteamento do kernel Linux utilizando `ip route` para verificar por qual caminho os pacotes destinados a redes externas são encaminhados.

```bash
ip route
```

![Consulta da tabela de roteamento com ip route](imagens/imagens%20aula%203/aula_3_IP_route.png)

A saída confirma a rota padrão (`default via 10.0.2.2`), indicando que todo o tráfego destinado para fora da sub-rede `10.0.2.0/24` é enviado ao gateway virtual `10.0.2.2` mantido pelo hipervisor.

---

### Passo 3 — Diagnóstico de conectividade por camadas (`ping`)

Para validar o funcionamento da pilha de protocolos TCP/IP de forma progressiva — do software local até a internet externa —, executei três testes de envio de pacotes ICMP (`ping -c 4`):

#### 3.1. Teste de conectividade local (Loopback)

Primeiro, testei a comunicação interna da própria pilha de rede do sistema operacional enviando pacotes para o endereço de *loopback* `127.0.0.1`.

```bash
ping -c 4 127.0.0.1
```

![Teste de conectividade local (Loopback) com ping](imagens/imagens%20aula%203/aula_3_Ping_1.png)

Os 4 pacotes foram transmitidos e recebidos com 0% de perda e tempo de resposta inferior a `0.1 ms`, confirmando que o stack TCP/IP do kernel está operacional.

#### 3.2. Teste de conectividade com o Gateway Padrão

Depois, testei a comunicação na rede local virtual enviando pacotes ICMP para o gateway padrão `10.0.2.2`.

```bash
ping -c 4 10.0.2.2
```

![Teste de conectividade com o Gateway padrão](imagens/imagens%20aula%203/aula_3_ping_2.png)

O gateway respondeu aos 4 pacotes sem perda (`0% packet loss`), comprovando a conectividade entre a VM e o roteador virtual do VirtualBox.

#### 3.3. Teste de conectividade externa com a Internet

Por fim, realizei um teste externo enviando pacotes para o servidor DNS público `8.8.8.8` para validar a tradução de endereços (NAT) e a saída para a internet.

```bash
ping -c 4 8.8.8.8
```

![Teste de conectividade externa com a Internet](imagens/imagens%20aula%203/aula_3_ping_3.png)

A resposta bem-sucedida dos 4 pacotes confirma que o VirtualBox traduziu corretamente o IP privado `10.0.2.15` para o IP da máquina física, garantindo acesso pleno à internet.

---

## 5. Testes, Comparativo NAT vs. Bridge e Tabela de Comandos

### Teste de Acessibilidade (Host $\rightarrow$ VM) e Comparativo NAT vs. Bridge

Após validar a saída da VM para a internet em modo **NAT**, realizei o teste inverso tentando alcançar a VM a partir do computador físico (host) e alternei o adaptador de rede para o **Modo Bridge** para comparar o comportamento de endereçamento e visibilidade na rede local:

| Critério de Comparação | Modo NAT (*Network Address Translation*) | Modo Bridge (Ponte) |
| :--- | :--- | :--- |
| **Endereço IP da VM** | Privado, gerado pelo DHCP interno do hipervisor (ex.: `10.0.2.15/24`). | Da mesma faixa da rede física local, fornecido pelo roteador físico da rede. |
| **Gateway Padrão** | Endereço interno reservado do hipervisor (`10.0.2.2`). | O roteador físico real da rede local. |
| **Host acessa a VM?** | **Não**, pois o host está fora da sub-rede privada NAT e o tráfego de entrada é bloqueado (salvo com regras de redirecionamento de portas). | **Sim**, pois a VM passa a atuar como um dispositivo independente visível na mesma rede física do host. |
| **Acesso à Internet** | Sim, por meio da tradução de endereços feita pelo hipervisor. | Sim, diretamente através do gateway da rede física. |

---

### Tabela de Comandos Utilizados

| Comando | O que significa e o que faz |
| :--- | :--- |
| `ip addr` | Exibe todas as interfaces de rede do sistema, seus estados (`UP`/`DOWN`), endereços físicos (MAC), endereços IPv4/IPv6 e máscaras de sub-rede. |
| `ip route` | Mostra a tabela de roteamento do kernel Linux, identificando as redes conhecidas e o endereço do gateway padrão (`default via ...`). |
| `ping -c 4 127.0.0.1` | Envia exatamente 4 pacotes ICMP (`-c 4`) para a interface de *loopback*, validando o funcionamento interno da pilha de rede do sistema operacional. |
| `ping -c 4 10.0.2.2` | Testa a comunicação de Camada 3 entre a máquina virtual e o gateway padrão da sub-rede NAT do VirtualBox. |
| `ping -c 4 8.8.8.8` | Verifica a conectividade externa e a tradução de endereços (NAT) até um servidor público na internet. |

---

## 6. Problemas e Soluções

* **Problema encontrado:** Com o adaptador de rede da VM configurado em **Modo NAT**, ao tentar executar um teste de `ping` a partir da máquina física (host) em direção ao endereço IP da máquina virtual (`10.0.2.15`), a comunicação falha e 100% dos pacotes são perdidos.
* **Diagnóstico:** O endereço `10.0.2.15` pertence a uma sub-rede privada isolada criada internamente pelo VirtualBox, para a qual o sistema operacional host não possui rota direta. O modo NAT mascara as conexões iniciadas de dentro da VM para fora, mas bloqueia conexões iniciadas externamente (do host para a VM) caso não exista uma regra explícita de *Port Forwarding*.
* **Solução aplicada:** Para permitir a comunicação bidirecional direta entre o host e a máquina virtual na mesma rede local, alterou-se a configuração do adaptador de rede no VirtualBox de **NAT** para **Modo Bridge**, fazendo com que a VM recebesse um IP válido diretamente do roteador físico e se tornasse acessível pela máquina hospedeira.

---

## 7. Conclusão

A prática permitiu consolidar na prática o funcionamento das camadas de rede — identificando o endereçamento físico na **Camada 2 (MAC)** e o endereçamento lógico e roteamento na **Camada 3 (IP e Gateway)**. Comprovou-se que o **modo NAT** é a escolha ideal quando o objetivo é apenas prover acesso rápido e isolado à internet para a máquina virtual sem consumir endereços da rede local, enquanto o **modo Bridge** é indispensável quando a VM precisa atuar como um servidor diretamente acessível pelo host e por outros dispositivos da rede física.
