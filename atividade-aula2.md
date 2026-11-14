# Relatório de Prática — Administração de Usuários, Grupos e Permissões no Linux

## 1. Identificação

* **Nome completo:** Max Clay da Silva
* **Turma:** 913-A
* **Data:** 18/10/2026
* **Título da prática:** Administração de Usuários, Grupos e Permissões no Linux

---

## 2. Objetivo

O objetivo desta aula prática foi administrar usuários, grupos e permissões de acesso a arquivos e diretórios no Ubuntu Server 26.04 LTS, aplicando o modelo de segurança do Linux baseado nas três esferas de propriedade: **Usuário Dono (`u`)**, **Grupo Associado (`g`)** e **Outros (`o`)**, combinadas às permissões de **Leitura (`r` / `4`)**, **Escrita (`w` / `2`)** e **Execução (`x` / `1`)**.

Ao longo da prática, foram executadas as seguintes atividades:
* Criação e gerenciamento de contas de usuários (`fulano`, `cicrano`, `beltrano` e `novato`) e senhas via terminal com `adduser`.
* Organização de usuários em grupos de trabalho (`devs` e `financeiro`) com `groupadd` e `usermod -aG`.
* Criação de diretórios compartilhados em `/srv/projeto` e `/srv/financeiro` e controle de posse com `chown` e `chgrp`.
* Manipulação de permissões em notação octal (`chmod 770` para diretórios e `chmod 660` para arquivos).
* Validação prática das restrições de acesso simulando diferentes perfis de usuários no terminal com `su -`.

---

## 3. Ambiente

* **Plataforma de virtualização:** Oracle VM VirtualBox
* **Máquina Virtual:** `ubuntu-server`
* **Sistema Operacional:** Ubuntu Server 26.04 LTS
* **Recursos alocados:** 768 MB de RAM e 1 vCPU
* **Usuários utilizados nos testes:**
  * `administrador`: conta administrativa com privilégios de `sudo` e proprietária dos diretórios compartilhados.
  * `fulano`: colaborador integrante apenas do grupo `devs`.
  * `cicrano`: colaborador integrante dos grupos `devs` e `financeiro`.
  * `beltrano`: colaborador integrante dos grupos `devs` e `financeiro`.
  * `novato`: usuário comum sem vínculo com nenhum grupo departamental (representando a esfera **Outros**).

---

## 4. Procedimento

### Passo 1 — Criação dos novos usuários no sistema e adicionando-os aos grupos

Primeiro, criei os usuários `fulano`, `cicrano`, `beltrano` e `novato`, usando `adduser`. Depois, para gerenciar permissões de forma escalável, criei o grupo `devs` com `groupadd` e associei os usuários `fulano`, `cicrano` e `beltrano` utilizando `usermod -aG`, mantendo `novato` fora do grupo para representar um usuário externo. Depois, validei a associação em `/etc/group`.

```bash
sudo adduser fulano
sudo adduser cicrano
sudo adduser beltrano
sudo adduser novato
sudo groupadd devs
sudo usermod -aG devs fulano
sudo usermod -aG devs cicrano
sudo usermod -aG devs beltrano
grep "devs" /etc/group
```

O comando `grep "devs" /etc/group` retornou `devs:x:1005:fulano,cicrano,beltrano`, comprovando que apenas os três desenvolvedores foram incluídos no grupo.

![Adicionamento dos usuários, grupos e colocamento dos usuários nos grupos](imagens/Imagens%20aula%202/aula_2_adicionando_usuarios_e_grupos.png)


![Demonstração do grep devs](imagens/Imagens%20aula%202/aula_2_grep_devs.png)

---

### Passo 2 — Criação do diretório de trabalho compartilhado (`/srv/projeto`)

Agora criei o diretório `/srv/projeto` para servir como repositório de trabalho da equipe e inspecionei suas propriedades e permissões iniciais com `ls -ld`.

```bash
sudo mkdir -p /srv/projeto
ls -ld
```

A saída `drwxr-xr-x 2 root root` mostrou que o diretório pertence inicialmente a `root:root` e concede permissão de leitura e entrada (`r-x`) tanto para o grupo quanto para qualquer outro usuário do sistema.

![Criando a pasta projeto,  e vendo com ls -ld](imagens/Imagens%20aula%202/aula_2_mkdir_projeto.png)

---

### Passo 3 — Mudança de dono e de grupo associado (`chown` e `chgrp`)

Para que o diretório `/srv/projeto` seja administrado pela conta `administrador` e compartilhado com a equipe `devs`, alterei o proprietário com `chown` e o grupo associado com `chgrp`.

```bash
sudo chown administrador /srv/projeto
sudo chgrp devs /srv/projeto
sudo chmod 770 /srv/projeto
ls -ld /srv/projeto
```
Para garantir controle total (`rwx` = `7`) ao dono (`administrador`) e ao grupo associado (`devs`), enquanto bloqueia completamente qualquer visualização ou entrada (`---` = `0`) de usuários externos, apliquei a máscara octal `770` com `sudo chmod 770`.

A listagem detalhada `ls -ld` passou a exibir `administrador devs`, confirmando a transferência de posse para o usuário `administrador` e o grupo `devs`.

![Aplicação da permissão octal 770 no diretório /srv/projeto e uso do chown/chgrp](imagens/Imagens%20aula%202/aula_2_chown_e_chgrp.png)

A saída de `ls -ld /srv/projeto` exibe `drwxrwx---`, confirmando que os bits de leitura, escrita e execução da esfera "outros" foram zerados e o diretório ficou isolado apenas para o dono e o grupo `devs`.

---

### Passo 4 — Criação e configuração de arquivo de teste (`config_redes.txt`)

Ainda como `administrador`, criei o arquivo `/srv/projeto/config-redes.txt` usando o comando `echo` com redirecionamento (`>`), entrei na pasta `/srv/projeto` e apliquei a permissão octal `660` (`rw-rw----`) para permitir que todos os integrantes do grupo `devs` leiam e editem o arquivo, sem conceder acesso a terceiros.

```bash
echo "Especificacao tecnica do roteador de borda" > /srv/projeto/config-redes.txt
ls -l /srv/projeto/config-redes.txt
chmod 660 /srv/projeto/config-redes.txt
cd /srv/projeto
ls -l
```

![Criação do arquivo config_redes.txt com echo e chmod](imagens/Imagens%20aula%202/aula_2_echo_file.png)

A listagem confirma que `config-redes.txt` possui permissões `-rw-rw----`, pertencendo a `administrador` e ao grupo `devs`, pronto para os testes de escrita colaborativa.

---

## 5. Testes, Exercício Prático e Tabela de Comandos

#### Links: https://github.com/mcs77-cyber/lsor-2026.2-sysdev-atividades-Max-Clay-da-Silva/tree/main/imagens/Imagens%20aula%202

### Teste A — Validação com usuário do grupo (`fulano`)

Neste teste, alternei para a sessão do usuário `fulano` utilizando `su - fulano`, naveguei até `/srv/projeto`, e adicionei uma nova linha ao arquivo `config-redes.txt` com `echo ... >>` para validar as permissões de leitura, escrita e execução do grupo `devs`.

```bash
su - fulano
cd /srv/projeto
echo "Revisado por Fulano" >> config-redes.txt
cat config-redes.txt
exit
```

![Teste A: acesso e leitura de /srv/projeto com fulano](imagens/Imagens%20aula%202/aula_2_acesso_e_leitura.png)

O usuário `fulano` acessou `/srv/projeto` e anexou a linha `"Revisado por Fulano"` sem restrições, comprovando que as permissões `770` na pasta e `660` no arquivo funcionam conforme o esperado para membros de `devs`.

---

### Teste B — Validação com usuário externo (`novato`)

Agora alternei para o usuário `novato` (que não pertence ao grupo `devs`) e tentei entrar no diretório `/srv/projeto` com `cd` e listar seu conteúdo com `ls -l` para validar o bloqueio da categoria "outros".

```bash
su - novato
cd /srv/projeto
ls -l /srv/projeto
exit
```

![Teste B: acesso negado ao diretório /srv/projeto para novato](imagens/Imagens%20aula%202/aula_2_su-novato%28e%20cd%20projeto%29.png)

O terminal exibiu `Permission denied` tanto na tentativa de navegação (`cd`) quanto na de listagem (`ls -l`), confirmando que o isolamento do diretório bloqueou completamente o acesso de usuários externos.

---

### Exercício Prático de Fixação — Departamento Financeiro (`/srv/financeiro`)

Para consolidar o controle de acesso departamental, criei o grupo `financeiro`, associei os usuários `cicrano` e `beltrano` a ele (mantendo-os também em `devs`), criei o diretório `/srv/financeiro`, transferi a posse para `administrador:financeiro` e apliquei a permissão restritiva `chmod 770`, por fim apliquei a permissão restritiva `chmod 660` para `/srv/financeiro/coisas.txt`.

```bash
sudo groupadd financeiro
sudo usermod -aG financeiro cicrano
sudo usermod -aG financeiro beltrano
sudo mkdir -p /srv/financeiro
sudo chown administrador:financeiro /srv/financeiro
sudo chmod 770 /srv/financeiro
ls -ld /srv/financeiro
chmod 660 /srv/financeiro/coisas.txt
ls -l /srv/financeiro/coisas.txt
```

![Exercicio pratico: groupadd, usermod e chown](imagens/Imagens%20aula%202/aula_2_exercicio_pratico.png)


![Exercicio pratico: sudo chmod 770](imagens/Imagens%20aula%202/aula_2_sudo_chmod_770.png)

Em seguida, validei o acesso permitido para `cicrano` (membro de `financeiro`) criando e editando um relatório financeiro, e o bloqueio de acesso para `fulano` (membro apenas de `devs`) e `novato` (sem grupos):

```bash
su - cicrano
cd /srv/financeiro
ls -l
echo "Algo, por cicrano" >> coisas.txt
cat coisas.txt
exit

# 2. Teste bloqueado com fulano (grupo devs)
su - fulano
cd /srv/financeiro
exit

# 3. Teste bloqueado com novato (outros)
su - novato
cd /srv/financeiro
exit
```

![Exercício Prático: acesso permitido para cicrano em /srv/financeiro](imagens/Imagens%20aula%202/aula_2_su-cicrano%28e%20cd%20financeiro%29.png)

![Exercício Prático: acesso bloqueado para fulano em /srv/financeiro](imagens/Imagens%20aula%202/aula_2_permissao_fulano.png)

![Exercício Prático: acesso bloqueado para novato em /srv/financeiro](imagens/Imagens%20aula%202/aula_2_su-novato%28depois%20da%20permissao%29.png)

Os resultados confirmam que `cicrano` conseguiu entrar em `/srv/financeiro`, criar e editar o relatório normalmente, enquanto tanto `fulano` quanto `novato` receberam `Permission denied`, garantindo o isolamento total entre os setores de desenvolvimento e financeiro.

---

### Tabela de Comandos Utilizados

| Comando | O que significa e/ou o que faz |
| :--- | :--- |
| `sudo adduser <usuario>` | Cria uma nova conta de usuário de forma interativa, gerando automaticamente sua pasta `/home/<usuario>`, definindo o shell `/bin/bash` e solicitando uma senha. |
| `tail -n 4 /etc/passwd` | Exibe as 4 últimas linhas do arquivo `/etc/passwd`, permitindo conferir as contas de usuários recém-criadas no sistema. |
| `sudo groupadd <grupo>` | Cria um novo grupo de trabalho no sistema operacional, registrando-o em `/etc/group`. |
| `sudo usermod -aG <grupo> <usuario>` | Modifica uma conta existente adicionando-a (`-a`, *append*) a um grupo secundário (`-G`) sem remover os grupos dos quais ela já participa. |
| `grep "<grupo>" /etc/group` | Filtra e exibe a linha do grupo específico dentro de `/etc/group`, mostrando seu GID e a lista de membros associados. |
| `sudo mkdir -p <caminho>` | Cria diretórios (*make directory*); a opção `-p` (*parents*) garante a criação de toda a estrutura de diretórios pais caso não existam. |
| `ls -l` / `ls -ld` | Lista detalhes como tipo, permissões (`rwx`), dono, grupo e data; a flag `-d` exibe os metadados do próprio diretório sem listar seu conteúdo interno. |
| `sudo chown <dono> <caminho>` | Altera o usuário proprietário (*change owner*) de um arquivo ou diretório (também permite alterar dono e grupo juntos com `dono:grupo`). |
| `sudo chgrp <grupo> <caminho>` | Altera o grupo associado (*change group*) a um arquivo ou diretório. |
| `sudo chmod 770` / `chmod 660` | Altera as permissões de acesso (*change mode*) em notação octal: `770` concede `rwx` ao dono e ao grupo e `---` a outros (pastas); `660` concede `rw-` ao dono e ao grupo e `---` a outros (arquivos). |
| `echo "texto" > arquivo` / `>>` | Escreve um texto na saída padrão e redireciona para um arquivo: `>` cria ou sobrescreve o conteúdo, enquanto `>>` anexa novas linhas ao final sem apagar o que já existe. |
| `su - <usuario>` | Alterna para a sessão de outro usuário (*substitute/switch user*); o hífen `-` carrega as variáveis de ambiente, o diretório pessoal e os grupos atualizados da conta destino. |

---

## 6. Problemas e Soluções

* **Problema encontrado:** Durante a execução dos testes de alternância de perfil no terminal, ao utilizar `su <usuario>` sem o hífen (`-`) logo após adicionar um colaborador a um novo grupo com `usermod -aG`, o sistema continuava negando acesso ao diretório departamental como se o vínculo de grupo não tivesse sido aplicado.
* **Solução:** O problema foi resolvido encerrando a sessão intermediária com `exit` e utilizando `su - <usuario>` (com hífen), obrigando o sistema operacional a iniciar uma sessão de login completa e recarregar a lista atualizada de grupos da conta testada.

---

## 7. Conclusão

A administração adequada de usuários, grupos e permissões no Linux é indispensável para assegurar a confidencialidade e a integridade das informações em ambientes corporativos multiusuário. Ao segmentar colaboradores em grupos funcionais (`devs` e `financeiro`) e aplicar permissões octais restritivas, impede-se que usuários externos ou de outros setores acessem ou modifiquem dados confidenciais, efetivando o princípio do menor privilégio no servidor.
