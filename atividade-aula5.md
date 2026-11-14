# Relatório de Prática — Estrutura FHS, Grupos e Permissões Departamentais no Linux

## 1. Identificação

* **Nome completo:** Max Clay da Silva
* **Turma:** 913-A
* **Data:** 14/11/2026
* **Título da prática:** Exploração do FHS, Criação de Diretórios Departamentais, Grupos e Controle de Permissões no Ubuntu Server

---

## 2. Objetivo

Nesta prática, foram exercitados os conceitos fundamentais de organização de arquivos e controle de acesso em sistemas Linux:
* Exploração da hierarquia padrão do sistema de arquivos (**FHS — Filesystem Hierarchy Standard**), identificando a função dos principais diretórios do sistema e verificando links simbólicos.
* Criação de estruturas de diretórios departamentais de forma recursiva utilizando o comando `mkdir -p`.
* Administração de grupos de usuários com `groupadd` e associação de contas existentes com `usermod -aG`.
* Configuração de propriedade (`chown`) e permissões restritivas para isolar o acesso entre setores distintos.
* Validação prática das regras de segurança por meio de testes de acesso autorizado e bloqueado alternando sessões de usuários com `su -`.

---

## 3. Ambiente

* **Plataforma de virtualização:** Oracle VM VirtualBox
* **Máquina Virtual:** `ubuntu-server`
* **Sistema Operacional:** Ubuntu Server 26.04 LTS
* **Recursos alocados:** 768 MB de RAM e 1 vCPU
* **Usuários utilizados nos testes:**
  * `administrador`: usuário administrativo com privilégios de `sudo` e proprietário das pastas departamentais.
  * `fulano`: colaborador pertencente ao grupo `ti-group`.
  * `cicrano`: colaborador pertencente ao grupo `vendas-group`.
  * `novato`: usuário comum sem vínculo com grupos departamentais.
  * `beltrano`: colaborador adicionado ao grupo do novo departamento criado no Desafio (`diretoria-group`).

---

## 4. Procedimento

### Passo 1 — Exploração do FHS e leitura de logs

Neste passo, navego pela raiz do sistema (`/`) e pelo diretório `/etc` para observar como o Linux organiza arquivos seguindo o padrão FHS — onde `/` é a raiz de toda a árvore, `/etc` armazena configurações globais do sistema, `/var` guarda dados variáveis como logs, `/srv` hospeda dados de serviços e `/usr` concentra binários e bibliotecas de programas. Também verifico que `/bin` e `/sbin` são links simbólicos apontando para `/usr` e leio as últimas linhas do log de autenticação.

```bash
ls /
ls -ld /bin /sbin
cd /etc
pwd /etc
ls -F | head -n 15
sudo tail -n 10 /var/log/auth.log
```
![Bin e Sbin](imagens/Imagens%20aula%205/aula_5_bin_e_sbin_.png)


![Bin e Sbin confirmando](imagens/Imagens%20aula%205/aula_5_bin_e_sbin_confirmacao.png)


![Exploração do FHS e logs de autenticação](imagens/Imagens%20aula%205/aula_5_15_itens_de_etc.png)


![Tail](imagens/Imagens%20aula%205/aula_5_tail_-n_10.png)

A saída mostra a estrutura padrão na raiz e confirma com `lrwxrwxrwx` que `/bin` e `/sbin` são links simbólicos apontando para `usr/bin` e `usr/sbin`. Além disso, `pwd` confirma a localização em `/etc`, `ls -F` diferencia arquivos de diretórios (com `/`) e o `auth.log` exibe os registros recentes de autenticação.

---

### Passo 2 — Criação da estrutura de diretórios departamentais

Agora vou criar os diretórios departamentais de TI e Vendas dentro de `/srv`, já incluindo subpastas internas em um único comando com `mkdir -p`, para organizar os arquivos de cada setor de forma isolada.

```bash
sudo mkdir -p ti-dept/projetos /srv/vendas-dept/relatorios
ls -R 
```

![Criação das pastas departamentais com mkdir -p](imagens/Imagens%20aula%205/aula_5_sudo-mkdir_ti-dept.png)
![ls -R](imagens/Imagens%20aula%205/aula_5_ls_-R.png)

O comando `ls -R` exibe recursivamente toda a árvore,incluindo (ti-dept, ti-dept/projetos, vendas-dept e vendas-dept/relatorios), comprovando que o `mkdir -p` criou tanto os diretórios pais quanto os subdiretórios sem erros.

---

### Passo 3 — Criação dos grupos e associação dos usuários

Neste passo, crio os grupos `ti-group` e `vendas-group` e adiciono os usuários `fulano` e `cicrano` aos seus respectivos setores usando `usermod -aG`, garantindo que eles mantenham seus grupos primários originais. Em seguida, confiro o vínculo com o comando `id`.

```bash
sudo groupadd ti-group
sudo groupadd vendas-group
sudo usermod -aG ti-group fulano
sudo usermod -aG vendas-group cicrano
id fulano
id cicrano
```

![Criação de grupos e verificação com id](imagens/Imagens%20aula%205/aula_5_criacao_dos_grupos_e_adicionamento_dos_usuarios.png)

A saída de `id fulano` mostra `ti-group` na lista de grupos secundários do usuário, enquanto `id cicrano` confirma sua inclusão em `vendas-group`, validando que os vínculos foram aplicados corretamente.

---

### Passo 4 — Donos e permissões das pastas

Antes de alterar as permissões, verifico os metadados atuais das pastas com `ls -ld`. Em seguida, transfiro a posse das pastas dos departamentos para o usuário `administrador` e para o grupo de cada departamento, aplicando a permissão `770` para que só o dono e os membros do grupo possam entrar e modificar arquivos na pasta.

```bash
ls -ld ti-dept vendas-dept
sudo chown -R administrador:ti-group ti-dept
sudo chown -R administrador:vendas-group vendas-dept
sudo chmod -R 770 ti-dept
sudo chmod -R 770 vendas-dept
ls -ld ti-dept vendas-dept
```

![Donos e permissões das pastas](imagens/Imagens%20aula%205/aula_5_donos_e_restricoes_das_pastas.png)

O primeiro `ls -ld` mostra que as pastas pertenciam inicialmente a `root:root` com permissão `drwxr-xr-x`. Após os comandos `chown` e `chmod`, a saída passa a exibir `drwxrwx---`, com dono `administrador` e o grupo de cada departamento, removendo qualquer acesso para outros usuários, como esperado.

---

### Passo 5 — Criação de arquivo departamental com permissão restrita (660)

Agora vou criar um arquivo interno dentro da pasta do setor de TI e definir sua propriedade para `administrador:ti-group` com permissão `660`, assegurando que apenas o proprietário e os membros da equipe de TI tenham permissão de leitura e escrita no documento, sem bit de execução.

```bash
sudo touch ti-dept/projetos/arquitetura_rede.txt
sudo chown administrador:ti-group ti-dept/projetos/arquitetura_rede.txt
sudo chmod 660 ti-dept/projetos/arquitetura_rede.txt
ls -l ti-dept/projetos/arquitetura_rede.txt
```

![Arquivo departamental com permissão 660](imagens/Imagens%20aula%205/aula_5_arquivo_confidencial.png)

A saída do `ls -l` apresenta `-rw-rw----` com dono `administrador` e grupo `ti-group`, confirmando que leitura e escrita estão restritas ao dono e ao grupo, sem acesso para terceiros.

---

## 5. Testes e Tabela de Comandos

### Teste A — Acesso permitido para membro do grupo (`fulano`)

Neste teste, inicio uma sessão completa como `fulano` usando `su -` para carregar corretamente seus grupos e tento acessar `/srv/ti-dept`, listar seu conteúdo e criar um arquivo de teste para confirmar que o acesso departamental funciona.

```bash
su - fulano
whoami
pwd /home/fulano
cd /srv/ti-dept
ls -l projetos/
echo "Rede do lab: 172.20.20.0/22" >> projetos/arquitetura_rede.txt
cat projetos/arquitetura_rede.txt
exit
```

![Teste A: acesso permitido para fulano](imagens/Imagens%20aula%205/aula_5_teste_A.png)

O comando `whoami` confirma o usuário `fulano`, que consegue entrar em `/srv/ti-dept` e usar `echo Rede do lab: 172.20.20.0/22" >> projetos/arquitetura_rede.txt`, e depois `cat projetos/arquitetura_rede.txt`, sem bloqueios.

---

### Teste B — Acesso bloqueado para usuário de outro departamento (`cicrano`)

Agora faço login como `cicrano` (membro apenas de `vendas-group`) e tento acessar a pasta `/srv/ti-dept` para validar se o isolamento entre departamentos impede a entrada e a leitura de arquivos por colaboradores de outros setores, podendo apenas acessar seu proprio setor.

```bash
su - cicrano
cd /srv/ti-dept
cd /srv/vendas-dept
ls
exit
```

![Teste B: acesso bloqueado para cicrano](imagens/Imagens%20aula%205/aula_5_teste_B.png)

O terminal retorna `Permission denied` ao tentar entrar em `/srv/ti-dept`, porém deixa entrar ao tentar entrar em `/srv/vendas-dept`, comprovando que `cicrano` está bloqueado pela permissão `770` em ti-dept, mas que ainda pode acessar vendas-dept.

---

### Teste C — Acesso bloqueado para usuário sem grupo departamental (`novato`)

Neste teste, acesso o sistema como `novato`, que não pertence nem a `ti-group` nem a `vendas-group`, e tento acessar ambas as pastas departamentais para confirmar que usuários gerais do servidor não possuem privilégios sobre nenhum setor.

```bash
su - novato
cd /srv/ti-dept
cd /srv/vendas-dept
exit
```

![Teste C: acesso bloqueado para novato](imagens/Imagens%20aula%205/aula_5_teste_C.png)

Em ambas as tentativas, o sistema exibe `Permission denied`, demonstrando que os outros estão bloqueados efetivamente de acessar qualquer departamento por estarem fora dos grupos autorizados.

---

### Desafio — Novo departamento (`financeiro-dept`) e testes com `beltrano` e `fulano`

Por fim, crio um terceiro departamento em /srv chamado `/diretoria-dept`), e seu respectivo grupo (`diretoria-group`). Adiciono o usuário `beltrano` a esse grupo, configuro dono e permissões e valido o acesso permitido para `beltrano` e o acesso para orcamento.txt

```bash
cd /srv 
sudo mkdir -p diretoria-dept
sudo groupadd diretoria-group
sudo usermod -aG diretoria-group beltrano
```
![Desafio: criação da diretoria](imagens/Imagens%20aula%205/aula_5_desafio_parte_1.png)
```
sudo chown -R administrador:diretoria-group diretoria-dept
sudo chmod -R 770 diretoria-dept
ls -ld diretoria-dept
sudo touch diretoria-dept/orcamento.txt
sudo chown administrador:diretoria-group diretoria-dept/orcamento.txt
sudo chmod 660 diretoria-dept/orcamento.txt
```
![Desafio: sudo chown e touch](imagens/Imagens%20aula%205/aula_5_desafio_parte_2.png)
```

su - beltrano
cd /srv/diretoria-dept
echo "Orcamento baixo, recursos necessarios imediatamente" >> orcamento.txt
cat orcamento.txt
exit
```
![Desafio: beltrano](imagens/Imagens%20aula%205/aula_5_desafio_parte_3(beltrano).png)
```
su - fulano
cd /srv/diretoria-dept
exit
```

![Desafio: fulano](imagens/Imagens%20aula%205/aula_5_desafio_parte_4(fulano).png)

A saída mostra que `beltrano` (membro de `diretoria-group`) acessa `/srv/diretoria-dept` e da echo no arquivo `orcamento.txt` normalmente, enquanto `fulano` recebe `Permission denied`, confirmando o isolamento total do novo departamento.

---

### Tabela de Comandos (Seção 6.4)

| Comando | O que significa e/ou o que faz |
| :--- | :--- |
| `cd /etc` | Altera o diretório de trabalho atual (*change directory*) para `/etc`, onde ficam os arquivos de configuração do sistema. |
| `pwd` | Exibe o caminho absoluto completo do diretório em que o usuário está no momento (*print working directory*). |
| `ls -F`, `ls -R`, `ls -ld` | Lista conteúdos: `-F` adiciona indicadores de tipo (como `/` para pastas e `@` para links); `-R` lista subdiretórios recursivamente; `-ld` exibe detalhes (permissões, dono, grupo) do próprio diretório sem listar seu interior. |
| `sudo tail -n 10 /var/log/auth.log` | Executa como superusuário (`sudo`) a leitura das últimas 10 linhas (`tail -n 10`) do arquivo de registro de autenticações e tentativas de acesso do sistema. |
| `mkdir -p` | Cria diretórios (*make directory*) e, com a opção `-p` (*parents*), cria automaticamente toda a hierarquia de pastas intermediárias caso ainda não existam, sem gerar erro se já existirem. |
| `groupadd` | Cria um novo grupo de usuários no sistema operacional. |
| `usermod -aG` | Modifica uma conta de usuário (`usermod`) adicionando-a (`-a`, *append*) a um ou mais grupos secundários (`-G`), sem remover os grupos aos quais ela já pertence. |
| `id` | Exibe o identificador numérico do usuário (UID), do seu grupo primário (GID) e a lista completa de todos os grupos dos quais ele faz parte. |
| `chown -R dono:grupo` | Altera o proprietário e o grupo (*change owner*) de um arquivo ou diretório; a flag `-R` aplica a mudança recursivamente a todos os subdiretórios e arquivos internos. |
| `chmod 770` / `chmod 660` | Altera as permissões de acesso (*change mode*) em notação octal: `770` dá leitura, escrita e execução (`rwx`) para dono e grupo e nada (`---`) para outros (ideal para pastas); `660` dá leitura e escrita (`rw-`) para dono e grupo e nada para outros (ideal para arquivos comuns). |
| `touch` | Cria um arquivo vazio caso ele não exista, ou atualiza os carimbos de data e hora (*timestamps*) de acesso e modificação de um arquivo existente. |
| `su -` x `su` | Alterna para outra conta de usuário (*switch user*): `su` troca apenas o usuário mantendo as variáveis de ambiente e o diretório atual da sessão anterior; `su -` (com hífen) inicia uma sessão de login completa, carregando o ambiente, variáveis e grupos atualizados do usuário destino. |
| `whoami` | Mostra o nome de login do usuário efetivo que está executando comandos na sessão atual do terminal. |

---

## 6. Problemas e Soluções

* **Problema encontrado:** Após adicionar um usuário a um grupo com `sudo usermod -aG`, ao testar o acesso na mesma sessão ou usando apenas `su usuario` (sem o hífen), o sistema continuou negando o acesso com `Permission denied`, como se o usuário ainda não fizesse parte do grupo.
* **Solução:** No Linux, a lista de grupos ativos de um usuário é carregada no momento em que a sessão de login é iniciada. O problema foi resolvido encerrando a sessão anterior com `exit` e iniciando uma nova sessão de login completa com `su - <usuario>`, fazendo com que o sistema recarregasse os grupos atualizados e liberasse o acesso esperado.

---

## 7. Conclusão

Separar diretórios por grupos departamentais e aplicar permissões restritivas (`770` em pastas e `660` em arquivos) é uma prática essencial para a segurança de servidores multiusuário porque materializa o **Princípio do Menor Privilégio**. Ao remover todas as permissões da categoria "outros" (`---`), o administrador garante que usuários comuns ou comprometidos em um setor não consigam visualizar, alterar ou apagar dados confidenciais de outros departamentos, preservando a confidencialidade, a integridade e a rastreabilidade das informações no ambiente corporativo.
