
<div align="center">

# PBL2 - Sistema de Bancos Distribuídos

</div>

<A name= "Intr"></A>

# Introdução
<p align='justify'>
Nos últimos anos, a adoção de movimentações financeiras exclusivamente por dispositivos móveis cresceu significativamente, impulsionada pela criação do Pix e pelo forte investimento dos bancos em aplicativos. Tal movimento transformou a maneira como os brasileiros realizam pagamentos, permitindo a inclusão financeira de milhões de pessoas. Inspirados pelo sucesso do Pix, o governo de um país sem banco central deseja desenvolver um sistema semelhante que permita a criação de contas bancárias, facilitando pagamentos, depósitos e transferências entre diferentes bancos sem a necessidade de um controle centralizado. Para isso, foi formado um consórcio bancário para desenvolver uma solução robusta e segura utilizando Python. Esta solução, através de um conjunto de estratégias, garante que as transações sejam atômicas e previne problemas como o duplo gasto, oferecendo uma infraestrutura confiável e eficiente para os clientes de todos os bancos participantes.
</p>

# Sumário
- <A href = "#Intr">Introdução</A><br>
- <A href = "#Api">Api</A><br>
- <A href = "#Interface">Interface Cli</A><br>
- <A href = "#Arq">Arquitetura da solução</A><br>
- <A href = "#Rest">Interface da Aplicação (REST)</A><br>
- <A href = "#Pro">Problemáticas e Soluções</A><br>
  - <A href = "#Trat">Problemáticas</A><br>
  - <A href = "#Sol">Soluções</A><br>
    - <A href = "#Rede">Rede em Anel (Token Ring)</A><br>
    - <A href = "#Lock">Lock </A><br>
    - <A href = "#2pc">2PC</A><br>
- <A href = "#clie">Utilizando a Interface</A><br>
- <A href = "#Teste">Testes</A><br>
- <A href = "#Exec">Como Executar</A><br>
- <A href = "#Res">Perguntas e Respostas</A><br>
- <A href = "#Conc">Conclusão</A><br>

<A name= "Api"></A>
# Api

<p align='justify'>
A API atua como servidor para um banco específico, permitindo aos usuários acessar diversas funcionalidades, tais como: criação de contas, realização de transferências, depósitos, saques, entre outras. Toda a comunicação ocorre por meio de uma API REST, implementada com Flask, uma biblioteca versátil da linguagem Python.
</p>

## Arquivo Principal (api.py)

<p align='justify'>
Este arquivo contém as rotas responsáveis pela gerenciamento de transações via token e pela realização de transações bancárias. Ele implementa métodos HTTP POST, GET, PUT e DELETE para operações específicas. Abaixo estão detalhadas as funcionalidades de cada rota e função desse arquivo.
</p>

 `Observação:` Vale ressaltar que a dinâmica de utilização do token para o gerenciamento de transações será abordada na subseção "Rede em Anel" da seção "Problemáticas e Soluções".
 
### Rotas

  - ***/id:*** Rota GET responsável por fornecer um ID válido para uso em um pacote de transações. Retorna o ID e o código 201 quando bem-sucedido.

  - ***/status:*** Rota GET responsável por verificar o status de um pacote de transações enviado para execução no banco, utilizando o ID da transação recebido no JSON, retorna o código 201 se o pacote foi executado com sucesso e 401 caso contrário.

  - ***/receive_token:*** Rota POST responsável por receber o token de outro banco do consórcio. Retorna o código 200 quando o token é recebido com uma sequência válida. A sequência estará presente no JSON.

  - ***/login:*** Rota POST responsável por verificar o usuário e senha enviados pelo JSON. Retorna o código 201 quando as credenciais correspondem aos dados do banco; caso contrário, retorna 401.

  - ***/users:*** Rota GET responsável por pegar os dados de todos os usuários do banco. Retorna 201 quando executado adequandamente. Vale informar que por segurança nenhum retorno da API retorna dados de senha.

  - ***/users/user_id:*** Rota GET responsável por pegar os dados de um usuário específico, através do user_id. Retorna 201 quando executado adequandamente e 401 caso contrário.

  - ***/users:*** Rota POST responsável por criar um usuário no banco. Retorna 201 quando executado adequandamente.

  - ***/users/user_id/accounts:*** Rota POST responsável por postar uma nova conta a um usuário existente, através do parametro user_id. Retorna 201 quando executado adequandamente e 401 caso contrário.

  - ***/users/user_id/accounts:*** Rota GET responsável por pegar todas as contas de um usuário específico, através do user_id. Retorna 201 quando executado adequandamente e 401 caso contrário.

  - ***/users/user_id/accounts/account_id:*** Rota GET responsável por pegar uma conta específico, através do user_id e account_id. Retorna 201 quando executado adequandamente e 401 caso contário.

  - ***/users/user_id:*** Rota DELETE responsável por deletar um usuário específico, através do user_id. Retorna 201 quando executado adequandamente.

  - ***/users/user_id:*** Rota PUT responsável por atualizar um usuário específico, através do user_id. Seu retorno é 201 para atualizado com sucesso e 401 caso contrário.

  - ***/users/user_id/accounts/account_id/deposit:*** Rota responsável por realizar um deposito em uma conta especificada pelo user_id e account_id. O JSON só indica o valor a ser depositado. Como as outras rotas, o retorno 201 representa que a operação foi bem sucedida e 401 representa o oposto.

  - ***/users/user_id/accounts/account_id/take:*** Rota responsável por realizar um saque em uma conta especificada pelo user_id e account_id. O JSON indica apenas o valor do saque. Assim como nas outras rotas, retorna o código 201 quando executado adequadamente e 401 caso contrário.

  - ***/sender:*** Rota responsável por adicionar o valor transferido à conta de destino, utilizando o ID e o valor recebidos do JSON. Ademais, retorna o código 201 se a operação for bem-sucedida e 401 caso contrário.

  - ***/abort:*** Rota responsável por reverter uma transação de um pacote. Ela utiliza do ID e valor recebido para realizar essa operação e retorna o código 201 se a reversão for bem-sucedida e 401 caso contrário.

  - ***/recipient:*** Rota responsável por receber um valor e um ID, e realizar o débito desse valor no saldo da conta associada a esse ID. Vale ressaltar que seu retorno é 201 quando executada corretamente e 401 caso contrário.

  - ***/transfers:*** Rota responsável por receber pacotes de transações da interface CLI a qualquer momento e armazená-los na lista de transferências. Se executada adequadamente, retorna o código 201; caso contrário, retorna 401.
     
### Funções

  - ***pass_token():*** Esta função opera em uma thread no servidor e tem como propósito transferir o token para outro banco dentro do consórcio. A transferência ocorre somente quando todas as condições necessárias para o repasse do token são satisfeitas.

  - ***receive_token():*** Esta função é responsável por executar uma sequência de operações imediatamente após receber o token pela rota designada. Primeiro, ela tenta realizar a operação. Em seguida, ela atualiza o valor de uma variável global, liberando uma thread encarregada de passar o token para outro banco do consórcio.

  - ***check_token():*** Esta função verifica o tempo que o banco está sem o token. Ademais, ela opera em uma thread separada para não interferir nas outras funções do servidor do banco e caso o banco fique sem o token por um período específico, a função interpreta que o consórcio perdeu o token e regenera um novo.

  - ***find_account(user, account_id):*** Esta função é responsável por buscar uma conta específica utilizando os parâmetros user e account_id. Retorna None caso a conta não exista no banco.

  - ***find_user(user_id):*** Função responsável por buscar um usuário específico na lista de usuários, utilizando o parâmetro user_id. Retorna None caso o usuário não seja encontrado.
  
  - ***abort_transactions(transactions):*** Esta função desempenha um papel crucial na garantia da atomicidade, sendo responsável por reverter as transações do pacote em caso de problemas durante sua execução. A rota "/abort" é utilizada para desfazer as transações que foram realizadas com sucesso anteriormente, e que estavam no mesmo pacote da transação que falhou.
    
  - ***transfer(transations):*** Tal função se caracteriza por executar todas as transações recebidas de maneira segura, garantindo a atomicidade dentro do pacote de transações passado como parâmetro desta função. Ademais, ao finalizar retorna True quando o pacote é executado com sucesso e False caso contrário.
 
  - ***init_server_flask():*** Esta função é responsável por inicializar a aplicação Flask.

  - ***main():*** Por último, esta função, como o próprio nome indica, é responsável pela ordem de execução das outras funções e rotas. Neste arquivo, ela é concisa, focando na inicialização das threads para passagem de token, verificação de token e da API do banco.

   `Observação:` Vale ressaltar que boa parte dessas funções possuem um bloco try-catch, visto que realizam operações delicadas. Mais detalhes podem ser encontrados na documentação do código.
    
<A name= "Interface"></A>
# Interface Cli

<p align='justify'>
A interface CLI permite que o usuário acesse as funcionalidades do banco. Para isso, o arquivo da interface utiliza requisições HTTP para interagir com o servidor do banco.
</p>

## Arquivo Principal (user.py)

<p align='justify'>
A seguir, uma breve descrição de cada função presente no arquivo 'user.py', que representa nossa interface CLI.
</p>

  - ***login(users):*** Esta função é responsável por realizar o login ou criar uma nova conta. No processo de login, utiliza a lista de usuários fornecida como parâmetro. O retorno são os dados do usuário logado.

  - ***createAccount(id, name, age, password, tipo, pix):*** Esta função é responsável por realizar a criação de uma conta dentro do banco. Os parâmetros da função representam os dados necessários para essa criação de conta.

  - ***verificarLogin(id, password):*** Esta função é responsável por verificar o login de um usuário em sua conta bancária, através de uma rota POST,  utilizando os parâmetros id e password.

  - ***addBank(id, saldo, password):*** Esta função tem como objetivo adicionar uma nova conta bancária a um usuário existente dentro do banco através de uma requisição POST. Os parâmetros incluem os dados dessa nova conta.

  - ***getUsers():*** Esta função é responsável por obter os dados de todos os usuário do banco. O retorno da função é a lista de usuários.

  - ***getUser(id):*** Esta função é responsável por obter os dados de um usuário do banco através de uma requisição GET. O parâmetro id é utilizado para buscar esse usuário no servidor do banco e o retorno é um dicionário com os dados do usuário.

  - ***logged(user):*** Esta função tem como propósito oferecer uma série de opções quando o usuário já estiver logado em sua conta no banco. As opções incluem: realizar transação, depósito, saque, sair, entre outras. O parâmetro user armazena os dados de conta do usuário.

  - ***verStatus(id):*** Esta função tem como objetivo realizar um GET no servidor do banco para verificar o status do último pacote de transações enviado pela interface. O parâmetro id representa o id do pacote de transações para fazer a verificação no banco.

  - ***getTransId():*** Esta função tem como objetivo realizar um GET de um ID válido no servidor do banco para adicionar a uma transação que está sendo criada. O retorno da função será esse ID.

  - ***requestSaque(valor, id, conta):*** Esta função é responsável por realizar o POST de um saque em uma conta especificada pelos parâmetros da função. Os parâmetros incluem o valor a ser sacado, o ID do usuário e a chave PIX da conta.

  - ***requestDeposito(valor, id, conta):*** Esta função é responsável por realizar o POST de um depósito em uma conta especificada pelos parâmetros da função. Os parâmetros incluem o valor a ser depositado, o ID do usuário e a chave PIX da conta.

  - ***requestTransferencia(dic, id_origem):*** Esta função tem como propósito realizar o POST de um pacote de transações no servidor do banco por meio de rotas HTTP. A rota utilizada é "/transfers". Os parâmetros da função representam o dicionário que será enviado e o id_origem, que identifica a conta que realizará o pacote de transferências.

  - ***limpar_terminal():*** Esta função tem como responsabilidade limpar o terminal. Para isso, ela verifica o sistema operacional para determinar qual função utilizar. Isto pois para limpar o terminal no "Windows" é diferente de limpar no "Linux".

  - ***main():*** Esta função é responsável por definir as regras de negócio do programa, estabelecendo a ordem de execução das funções e procedimentos presentes no sistema.

   `Observação:` Vale ressaltar que boa parte dessas funções possuem um bloco try-catch, visto que realizam operações delicadas. Mais detalhes podem ser encontrados na documentação do código.

<A name="Arq"></A>
# Arquitetura da solução

<p align="justify">
Para abordar a arquitetura utilizada na interação entre bancos e contas, onde as transações são realizadas de maneira descentralizada, é instrutivo primeiro explicar como essas transações são normalmente efetuadas de modo centralizado, com o auxílio do banco central. Em seguida, podemos explorar a arquitetura descentralizada adotada neste sistema, assim como as soluções implementadas para mitigar os problemas inerentes aos sistemas descentralizados.
</p>

### Sistema de Bancos Centralizado

<p align="justify">
Um sistema bancário centralizado envolve o banco central desempenhando um papel crucial na intermediação e controle das transações entre instituições financeiras. Nesse modelo, todas as transações, como transferências de fundos entre bancos e liquidação de pagamentos, são processadas e monitoradas pelo banco central. Isso implica que todas as partes envolvidas dependem diretamente das operações e diretrizes estabelecidas pelo banco central para as suas atividades financeiras. Esse sistema centralizado proporciona um controle rigoroso e uma supervisão eficiente das operações financeiras, mas também pode apresentar desafios como possíveis gargalos de comunicação e vulnerabilidades à falhas sistêmicas.
</p>
  
- ***Banco -> Banco e Banco -> Banco Central:*** A comunicação entre bancos para o envio de diversas transações pode ser realizada utilizando API REST, semelhante a qualquer outro tipo de API. As regras e protocolos de comunicação são determinados pelo banco intermediador da transação, que, neste caso, é o banco central.

  `Observação:` Qualquer falha no Banco Central afeta todos os bancos envolvidos, o que também se aplica a falhas de segurança.
   
  `Definição:` Uma API REST (Representational State Transfer) é uma arquitetura de comunicação que utiliza os princípios do protocolo HTTP para permitir a comunicação entre sistemas distribuídos.

Abaixo você pode ver como se da a interação entre os bancos e o banco cantral, para isso observe a <em>Figura 1.</em> 

 <div align="center">
   
   ![Figura 1](Images/Diagrama.png)
   <br/> <em>Figura 1. Sistema de Bancos Centralizado.</em> <br/>
   
   </div>

<p align="justify">
Analisando a imagem, observa-se que toda a comunicação depende do agente central. Em outras palavras, qualquer interação entre os bancos necessariamente envolve o banco central. No entanto, um ponto relevante ao utilizar um sistema bancário centralizado é que ele tende a apresentar menos problemas relacionados à concorrência, e quando surgem problemas, as soluções são geralmente mais simples do que em sistemas distribuídos.
</p>

### Sistema de Bancos Descentralizados (Utilizado nesta Aplicação)

<p align="justify">
 Em um sistema de bancos distribuídos, a arquitetura permite que múltiplos bancos operem de forma interligada e colaborativa, sem depender exclusivamente de um ponto central de controle. Cada banco possui autonomia para processar transações localmente e se comunicar diretamente com outros bancos dentro do consórcio. Essa abordagem promove maior resiliência e escalabilidade ao sistema, uma vez que falhas em um banco não comprometem necessariamente todo o sistema. Além disso, a distribuição de recursos e responsabilidades entre os bancos pode otimizar a eficiência operacional e aumentar a segurança, permitindo o processamento paralelo e redundante das transações, reduzindo a vulnerabilidade a falhas em pontos únicos. 
</p>

`Observação:` Ademais, note que sistemas distribuídos possuem vários desafios na sua implementação. Gerenciar a consistência dos dados entre diferentes bancos, garantir a sincronização adequada das transações, lidar com a latência das comunicações entre os bancos e garantir a segurança de ponta a ponta são alguns dos desafios críticos enfrentados. Além disso, a complexidade aumenta com a necessidade de implementar algoritmos distribuídos para coordenação e consenso entre os sistemas, visando evitar problemas como inconsistências de dados e conflitos de transações. Tais problemáticas sobre a implementação desse sistema serão abordadas na seção "Problemáticas e Soluções".

 <div align="center">
   
   ![Figura 1](Images/Diagrama2.png)
   <br/> <em>Figura 2. Sistema de Bancos Descentralizados.</em> <br/>
   
   </div>

<p align="justify">
Observe que a imagem ilustra que um banco pode se conectar com todos os outros bancos presentes no consórcio. Isso ocorre porque, em caso de falha no sistema de um desses bancos, ele pode tentar se comunicar com outro banco disponível. Dessa forma, ao contrário dos sistemas centralizados, a dependência é reduzida, pois para que todo o sistema pare, todos os bancos teriam que estar fora do ar simultaneamente.
</p>

<A name="Rest"></A>
# Interface da Aplicação (REST)

Em relação a API foram criadas diversas rotas, as quais utilizaram verbos/métodos como POST, PUT, GET e DELETE.

- ***POST:*** Em relação aos métodos POST, temos dez rotas que à utilizam. Podemos ver a estrutura para construção dessas rotas na Figura 3, 4, 5, 6, 7 e 8.
  
  <div align="center">
   
   ![Figura 2](Images/POST0.png)
   <br/> <em>Figura 3. Métodos POST.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 2](Images/POST1.png)
   <br/> <em>Figura 4. Métodos POST.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 2](Images/POST2.png)
   <br/> <em>Figura 5. Métodos POST.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 2](Images/POST3.png)
   <br/> <em>Figura 6. Métodos POST.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 2](Images/POST4.png)
   <br/> <em>Figura 7. Método POST.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 2](Images/POST5.png)
   <br/> <em>Figura 8. Método POST.</em> <br/>
   
   </div>

- ***PUT:*** Só há uma rota que utiliza este método, tal rota se chama "http://{id do banco junto com a porta}/users/{id do usuário}". Vale lembrar que essa método realiza atualizações na minha API. Mais detalhes na Figura 9.

<div align="center">
   
   ![Figura 3](Images/PUT.png)
   <br/> <em>Figura 9. Método PUT.</em> <br/>
   
   </div>

- ***GET:*** Já em relação aos métodos GET utilizados, temos 6, os quais são dois para transações e 4 para usuários e contas. As referêntes a transação são as rotas "http://{id do banco junto com a porta}/id" e "http://{id do banco junto com a porta}/status". Já as outras rotas são referêntes as contas e usuários do banco, sendo elas: "http://{id do banco junto com a porta}/users", "http://{id do banco junto com a porta}/users/{id do usuário}", "http://{id do banco junto com a porta}/users/{id do usuário}/accounts" e "http://{id do banco junto com a porta}/users/{id do usuário}/accounts/{id da conta}" , respectivamente. Podemos ver mais detalhes nas Figuras 10, 11 e 12.

<div align="center">
   
   ![Figura 4](Images/GET0.png)
   <br/> <em>Figura 10. Métodos GET.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 4](Images/GET1.png)
   <br/> <em>Figura 11. Métodos GET.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 4](Images/GET2.png)
   <br/> <em>Figura 12. Métodos GET.</em> <br/>
   
   </div>

- ***DELETE:*** Por fim os métodos DELETE, temos somente 1, assim como os método PUT, , cuja rota é "http://{id do banco junto com a porta}/users/{id do usuário}". Para mais detalhes observe a Figura 13.

<div align="center">
   
   ![Figura 5](Images/DELETE.png)
   <br/> <em>Figura 13. Métodos DELETE.</em> <br/>
   
   </div>

<A name= "Pro"></A>
# Problemáticas e Soluções 

<A name="Trat"></A>
# Problemáticas

Desenvolver uma aplicação de bancos distribuídos apresenta uma série de desafios e problemáticas que precisam ser cuidadosamente considerados:

- **Consistência dos Dados:** Garantir que todos os nós do sistema tenham uma visão consistente dos dados em um ambiente distribuído pode ser complexo. Decidir entre consistência forte e eventual pode afetar diretamente o desempenho e a integridade dos dados.

- **Concorrência e Conflitos:** Conflitos de escrita podem ocorrer quando múltiplos nós tentam atualizar os mesmos dados simultaneamente. Implementar estratégias de controle de concorrência, como locking ou versões de dados, é essencial para mitigar esse problema.

- **Tolerância a Falhas:** Em um ambiente distribuído, falhas de rede, falhas de nó e falhas de comunicação são mais comuns. Implementar mecanismos robustos de detecção de falhas e recuperação é crucial para manter a disponibilidade do sistema.

- **Coordenação de Transações:** Coordenar transações distribuídas entre múltiplos nós sem comprometer a consistência ou a disponibilidade dos dados é um desafio significativo. Protocolos de transação como o Two-Phase Commit (2PC) ou mecanismos alternativos como transações compensatórias são frequentemente usados para resolver esse problema.

- **Deadlocks:** Ocorre quando duas ou mais threads ficam aguardando indefinidamente por recursos que a outra possui. Isso pode paralisar o sistema.

- **Livelocks:** É uma situação em sistemas distribuídos ou concorrentes onde dois ou mais processos ou threads ficam presos em um ciclo de interação contínua, sem conseguir progredir. Ao contrário de um deadlock, onde os processos estão esperando recursos que estão sendo mantidos por outros processos, no livelock os processos estão ativos e tentando resolver uma condição de corrida ou de concorrência de forma ineficaz.

<A name="Sol"></A>
# Soluções

<p align="Justify">
Este projeto aplicou diferentes soluções para enfrentar os desafios mencionados anteriormente. Foram utilizados locks para lidar com a concorrência entre contas dentro do mesmo banco. Além disso, foi implementada uma estrutura de rede em anel para o consórcio, o que permitiu atribuir uma coordenação e ordem às transações entre diferentes bancos, evitando conflitos de operações externas que poderiam interferir nas operações de outros bancos.

Para garantir a atomicidade das transações no pacote de transações, foi empregado o protocolo Two-Phase Commit (2PC). Para mais detalhes sobre a implementação dessas soluções, é possível analisar o código-fonte do projeto. Abaixo, vou discorrer um pouco mais sobre cada uma dessas abordagens.
</p>

<A name= "Rede"></A>
# Rede em Anel (Token Ring)

O Token Ring é uma arquitetura de rede de computadores que utiliza um token (ou ficha) para controlar o acesso aos recursos compartilhados entre os dispositivos conectados. Abaixo estão os principais pontos sobre o Token Ring:

### Funcionamento do Token Ring:

1 - Topologia de Rede:

  - Os dispositivos são organizados em um anel físico ou lógico fechado, onde cada dispositivo está conectado diretamente aos seus vizinhos imediatos.
A transmissão de dados ocorre sequencialmente de um dispositivo para o próximo, formando um caminho unidirecional ao redor do anel.

2 - Token Passante:

  - Um token (ou ficha) circula continuamente ao redor do anel, permitindo que dispositivos transmitam dados apenas quando possuem o token.
O dispositivo que possui o token tem permissão exclusiva para enviar dados, enquanto os demais dispositivos apenas encaminham o token para o próximo na sequência.

3 - Controle de Acesso ao Meio:

  - O Token Ring oferece um método eficiente para controlar o acesso ao meio de transmissão (o anel), minimizando colisões de dados e garantindo um fluxo ordenado de comunicação.

### Vantagens do Token Ring:

  - Baixa Colisão de Dados: Como o token controla o acesso ao meio, há menos probabilidade de colisões de dados em comparação com outras topologias.

  - Eficiência em Ambientes de Tráfego Pesado: O Token Ring pode ser mais eficiente do que Ethernet em ambientes onde há um alto volume de tráfego de dados, devido à sua capacidade de gerenciar o acesso ao meio de forma ordenada.

`Observação:` Nesta aplicação, são implementadas estratégias inspiradas na estrutura do Token Ring, incluíndo até a regeneração do token na rede, em certas circunstâncias.

<A name= "Lock"></A>
# Lock


O uso de locks (ou travas) é fundamental em programação concorrente para garantir a consistência e a integridade dos dados compartilhados entre threads ou processos.

### Propósito e Funcionamento:

  - Garantia de Exclusão Mútua: O principal objetivo de um lock é garantir que apenas uma thread por vez tenha acesso a um recurso compartilhado (como uma variável, uma seção crítica do código ou um banco de dados), evitando assim condições de corrida.

  - Operação Atômica: Quando uma thread adquire um lock, ela ganha o direito exclusivo de acessar o recurso protegido. Isso garante que nenhuma outra thread possa acessar ou modificar o recurso até que o lock seja liberado.

`Observação:` É importante mencionar que o uso de locks nesta aplicação poderia potencialmente causar deadlocks. No entanto, devido à utilização da rede em anel, esse problema é mitigado.

<A name= "2pc"></A>
# 2PC

### Two-Phase Commit (2PC):

Objetivo: Garantir a atomicidade das transações em sistemas distribuídos.

  - Fase 1 (Preparação): O coordenador envia uma solicitação de preparação para todos os participantes. Cada participante responde com um voto de "preparado" ou "aborto" com base na sua capacidade de executar a transação com segurança.

  - Fase 2 (Confirmação): Se todos os participantes estiverem preparados, o coordenador envia uma mensagem de commit. Todos os participantes então executam a transação de forma definitiva (commit). Se algum participante não puder prosseguir, o coordenador envia uma mensagem de rollback para abortar a transação.

`Observação:` Sobre o 2PC, vale informar que caso a confirmação demonstre falha, a preparação será desfeita.

<A name="clie"></A>
# Utilizando a Interface

Observe que abaixo seguem algumas Figuras que mostram como a interface CLI se comporta.

<div align="center">
   
   ![Figura 14](Images/cli0.png)
   <br/> <em>Figura 14. Interface Cli para Criação de Conta ou Login.</em> <br/>
   
   </div>

  <div align="center">
   
   ![Figura 15](Images/cli1.png)
   <br/> <em>Figura 15. Criação de Conta.</em> <br/>
   
   </div>

<div align="center">
   
   ![Figura 16](Images/cli7.png)
   <br/> <em>Figura 16. Login em Conta.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 17](Images/cli2.png)
   <br/> <em>Figura 17. Interface Cli Quando Logado.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 18](Images/cli3.png)
   <br/> <em>Figura 18. Tentativa de Depósito.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 19](Images/cli4.png)
   <br/> <em>Figura 19. Visualização do Saldo após Depósito.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 20](Images/cli5.png)
   <br/> <em>Figura 20. Tentativa de Saque.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 21](Images/cli6.png)
   <br/> <em>Figura 21. Visualização do Saldo após saque e Tentativa de Criação de um Pacote de Transações.</em> <br/>
   
   </div>
   
`Observação:` Para essa seção não se estender muito, na parte da interface referente ao pacote de transações, quando digitar 1, você irá adicionar outra transação ao pacote; e, se digitar 2, o pacote de transações será enviado para ser executado pelo servidor do banco.

<A name="Teste"></A>
# Testes

<p align="justify">
A seguir, serão apresentados uma série de testes realizados utilizando as rotas, utilizando ferramentas como Postman e Thunder Client. Além de visualizar o terminal do servidor.
</p>

<div align="center">

<p align="justify">
  Teste nº 1 - As Figuras 22 e 23 ilustram a correta passagem do token. Note que a sequência que chega a um servidor é sempre diferente daquela que chega a outro servidor. Esse teste foi realizado para evitar a duplicação do token na rede.
  </p>
   
   ![Figura 22](Images/Token1.png)
   <br/> <em>Figura 22. Visualização da Passagem de Token pelos Bancos do Consórcio.</em> <br/>
   
   </div>

  <div align="center">
   
   ![Figura 23](Images/Token0.png)
   <br/> <em>Figura 23. Visualização da Passagem de Token pelos Bancos do Consórcio.</em> <br/>
   
   </div>

<p align="justify">
  Teste nº 2 - A Figura 24 ilustra a reeleição do token na rede quando ele se perde. Considera-se o token perdido quando um servidor fica sem recebê-lo por um determinado período de tempo.
</p>

<div align="center">
   
   ![Figura 24](Images/ReeleicaoToken.png)
   <br/> <em>Figura 24. Visualização da Reeleição do Token na rede quando ele se perde.</em> <br/>
   
   </div>

<p align="justify">
  Teste nº 3 - As Figuras 25 e 26 ilustram os dados das contas antes da execução dos testes mostrados nas Figuras 27 e 28, que foram realizados simultaneamente utilizando o Flows do Postman, conforme exibido na Figura 29. Esse teste foi realizado para observar se um dado está sendo sobreposto pelo outro. Os resultados podem ser vistos nas Figuras 30 e 31. Ao analisar sa gravuras irá perceber que isso não ocorre.
</p>

  <div align="center">
   
   ![Figura 25](Images/Status0.png)
   <br/> <em>Figura 25.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 26](Images/Status1.png)
   <br/> <em>Figura 26.</em> <br/>
   
   </div>

<div align="center">
   
   ![Figura 27](Images/req1.png)
   <br/> <em>Figura 27.</em> <br/>
   
   </div>

<div align="center">
   
   ![Figura 28](Images/req2.png)
   <br/> <em>Figura 28.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 29](Images/Flows.png)
   <br/> <em>Figura 29.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 30](Images/Status2.png)
   <br/> <em>Figura 30.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 31](Images/Status3.png)
   <br/> <em>Figura 31.</em> <br/>
   
   </div>

<p align="justify">
  Teste nº 4 - Foi realizado outro teste utilizando os dados iniciais ilustrados nas Figuras 30 e 31. Desta vez, os dados mostrados na Figura 27 foram enviados duas vezes para verificar se a operação seria realizada mesmo sem saldo. Os resultados podem ser vistos nas Figuras 32 e 33. Ao observar o resultado perceberá que não houve duplo abate, já que isso iria deixar a conta com saldo negativo.
</p>

   <div align="center">
   
   ![Figura 32](Images/Status4.png)
   <br/> <em>Figura 32.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 33](Images/Status5.png)
   <br/> <em>Figura 33.</em> <br/>
   
   </div>

<p align="justify">
  Teste nº 5 - As Figuras 32 e 33 representam os dados iniciais para o teste de um pacote de transferências que será executado pelo servidor. O pacote pode ser visualizado na Figura 34. Neste caso, a primeira transação será bem-sucedida; no entanto, devido à impossibilidade de execução da segunda transação do pacote, o pacote inteiro será abortado, resultando na reversão da primeira transação. Este teste serviu para ver se a reversão do pacote está funcionando adequadamente. Os resultados podem ser vistos novamente nas Figuras 32 e 33.
</p>

   <div align="center">
   
   ![Figura 34](Images/FalhaPacote.png)
   <br/> <em>Figura 34.</em> <br/>
   
   </div>

<p align="justify">
  Teste nº 6 - Por último, foi realizado um teste utilizando novamente os dados ilustrados nas Figuras 32 e 33. O pacote a ser enviado está ilustrado na Figura 35. O objetivo deste teste é verificar se o pacote será executado corretamente, uma vez que todas as contas possuem fundos para realizar as operações. Os resultados desse teste podem ser visualizados na Figura 36 e 33.
</p>

   <div align="center">
   
   ![Figura 35](Images/PacoteValido.png)
   <br/> <em>Figura 35.</em> <br/>
   
   </div>

   <div align="center">
   
   ![Figura 36](Images/Status6.png)
   <br/> <em>Figura 36.</em> <br/>
   
   </div>

`Observação:` Há uma variedade de outros testes que poderiam ser demonstrados nesta seção. No entanto, para não se estender muito, ela focou em mostrar os testes mais relevantes. Para mais testes, você pode verificar ao executar esta aplicação.

<A name="Exec"></A>
# Como Executar

## Etapas:

### 1. Configuração do Ambiente:

   - **Requisitos do Sistema:** Será preciso ter ao menos o Docker instalado na máquina para que seja possível criar a imagem e executá-la.

`Observação:` Caso não possua o Docker instalado você poderá executar os arquivos api.py e user.py via terminal. 
     
### 2. Obtenção do Código Fonte:

   - **Clonagem do Repositório:** Você pode utilizar o seguinte comando no terminal para adquirir a aplicação:                                          

           git clone https://github.com/Emanuel-Antonio/PBL2-Redes.git.
     
   - **Download do Código Fonte:** Caso não tenha o Git na máquina, você pode fazer o download desse repositório manualmente. Vá até o canto superior, selecione "Code" e depois "Download ZIP", e então extraia o arquivo ZIP na sua máquina.

### 3. Configuração da Aplicação:

   - **Arquivos de Configuração:** Abra as pastas "Bank" e "Client" e altere nos arquivos "api.py" e "user.py" o endereço IP para o endereço da máquina onde o banco está rodando, banco é representado pelo arquivo api.py.
     
`Observação:` Execute essa etapa somente se não informar o IP ao rodar a imagem do docker.  

### 4. Execução da Aplicação:

   - **Com Docker:**
     
     1. Execute o seguinte comando no terminal dentro das pastas Bank e Client: "docker build -t nome_do_arquivo .", para gerar as imagens, repita duas vezes.
        
     2. Agora execute as imagens usando o comando "docker run --network='host' -it -e IP=ipBank nome_da_imagem" para executar as imagens do api.py e user.py.

`Observação:` Se não quiser rodar através do docker execute pelo terminal usando o comando "python arquivo.py", para os dois arquivos antes mencionados.

<A name="Res"></A>
# Perguntas e Respostas

 - Esta Aplicação Permite gerenciar contas ?

    `Resposta:` Sim, esta aplicação permite realizar a criação, atualização, aquisição e deleção de contas. Isso fica evidente nas seções Interface da Arquitetura (REST), Testes e Utilizando a Interface. Ademais, isto também vale para a criação e realização de transações.

 - Permite selecionar e realizar transferência entre diferentes contas?

    `Resposta:` Sim, você pode ver um teste que aborda essa situação na seção Testes, mais precisamente no teste nº 3.

 - Comunicação entre servidores

    `Resposta:` Ocorre de maneira adequada, utilizando API REST, a verificação pode ser feita na seção Interface da Arquitetura (REST).

 - Sincronização em um único servidor

    `Resposta:` Foi tratado através do uso de locks, e o 2PC auxilia na atomicidade das transações. Você pode ver mais informações sobre essas estratégias na seção Problemáticas e Soluções. Além disso, após uma série de testes, essa solução, em conjunto com uma API REST implementada através da biblioteca Flask, mostrou-se eficiente.

 - Algoritmo da concorrência distribuída está teoricamente bem empregado?

    `Resposta:` A estratégia utilizada para resolver a problemática de concorrência distribuída foi implementar o sistema na estrutura de token ring. Esta é uma das soluções possíveis para problemas de concorrência em sistemas distribuídos, embora não seja a mais eficiente. Há mais informações sobre o Token Ring na seção Problemáticas e Soluções.

 - Algoritmo está tratrando o problema na prática?

    `Resposta:` Sim, pode verificar o pleno funcionamento na seção Testes, caso persista alguma dúvida sobre o funcionamento pode verificar o código fonte.

 - Tratamento da confiabilidade

    `Resposta:` Quanto à confiabilidade do programa, ele é confiável: quando um banco se desconecta, há mecanismos para evitar que todo o sistema seja afetado. Uma das estratégias utilizadas é a regeneração do token. Quando o banco retorna, há uma verificação para garantir que não existam múltiplos tokens na rede. Caso haja, o sistema invalidará os tokens excedentes antes de executar qualquer operação. Tem algumas informações adicionais na seção Problemáticas e Soluções.

 - Pelo menos uma transação concorrente é realizada ?

    `Resposta:` Sim, isso é garantido pelo uso de Locks e o uso de uma fila de pacotes de transações. Assim, cada transação ocorre de maneira isolada e sequencial, uma por vez. Portanto, se uma transação requer que dinheiro seja recebido para realizar outra operação, isso dependerá da ordem das operações dentro da lista de pacotes de transações de cada banco. Além disso, através de testes, verificou-se o pleno funcionamento em relação a essa problemática. As contas apresentaram os saldos corretamente de acordo com a ordem das transações, garantindo que os clientes possam realizar suas transações de maneira adequada, e sua conclusão dependerá somente do saldo bancário. Para conferir existe um teste sobre esse possível problema na seção Testes.

 - Uso do docker

    `Resposta:` Esta aplicação faz uso do docker. Isto é evidenciado na seção Como Executar.

 - Documentação do código (Github)

    `Resposta:` A aplicação está devidamente comentada no seu código-fonte, assim como os commits do projeto seguem um padrão claro e compreensível.


<A name="Conc"></A>
# Conclusão

<p align='justify'>
Por fim, destaco que este projeto atende a todas as exigências previamente propostas e desempenha um papel significativo no aprimoramento das habilidades na área de concorrência e conectividade. No entanto, há espaço para melhorias futuras, como o desenvolvimento de uma interface web ou mobile.
</p>
