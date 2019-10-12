# RASA ChatBot ASHA
<!-- badges -->
<a href="https://www.gnu.org/licenses/gpl-3.0.pt-br.html"><img src="https://img.shields.io/badge/licence-GPL3-green.svg"/></a>
<a href="https://codeclimate.com/github/lappis-unb/rasa-ptbr-boilerplate/maintainability"><img src="https://api.codeclimate.com/v1/badges/3fe22bf52000e147c6df/maintainability"/></a>


Esse projeto de Inteligencia Artificial está sendo feito em PYTHON com a biblioteca RASA NLU e RASA CORE com configurações necessárias para a construção de um projeto de ChatBot com objetivo de ajudar pessoas em depressão com objetivo de iniciar uma conversa, e se necessário, encaminhar para o atendimento humano do Disk-Vida.

Orgulhosamente esse projeto teve como base o [Boilerplate](https://github.com/lappis-unb/rasa-ptbr-boilerplate) do Laboratório Avançado de Produção, Pesquisa e Inovação em Software (LAPPIS/UnB)

# Entenda a Arquitetura

É utilizado no boilerplate diversas tecnologias que interagem entre si para obter um melhor resultado. Veja a arquitetura implementada:

![](https://user-images.githubusercontent.com/8556291/57933140-d8d66b80-7892-11e9-8d58-a7eda60b099b.png)

O usuário interage com o Bot via ROCKETCHAT, TELEGRAM ou MESSENGER, que envia as mensagens para o RASA NLU através de conectores, onde ele identifica a *intent*, e responde pelo RASA CORE, de acordo com as *stories* e *actions*.  
As *models* utilizadas para a conversação são geradas pelo módulo *trainer* e depois transferidas para o Bot. Estes *models* podem ser versionados e evoluídos entre bots.  
O NOTEBOOK JUPITER avalia o funcionamento de acordo com o formato das *intents* e das *stories*, gerando gráficos.
O ELASTICSEARCH coleta os dados da conversa e armazena para a análise feita pelo KIBANA, que gera gráficos para
avaliação das conversas dos usuários e do boilerplate.

## Bot

Este script foi configurado para construir as imagens genéricas necessárias para execução deste ambiente.
Caso seu projeto utilize este boilerplate e vá realizar uma integração contínua ou similar, é interessante
criar um repositório para as imagens e substituir os nomes das imagens "asha Bot", "asha/coach" e "asha/botrequirements" pelas suas respectivas novas imagens, por exemplo "<organização> Bot" em repositório público.

### ROCKETCHAT
Para testar o assistente virtual utilizando da plataforma do ROCKETCHAT, siga os seguintes comandos para subir os containers em seu computador:

```sh
sudo docker-compose up -d rocketchat

# Caso não vá subir a stack do Analytics
# aguarde o container do rocketchat subir

sudo docker-compose up -d Bot
```

Caso queira subir toda a stack e vá utilizar do analytics é necessário substituir a seguinte variável de ambiente no `docker Bot-rocketchat.env` de **False** para **True**.
```
# Analytics config
ENABLE_ANALYTICS=True

# E logo após trocar, subir o container do Bot
sudo docker-compose up -d Bot
```

Após esses comandos o ROCKETCHAT deve estar disponível na porta 3000 do seu computador. Entre em http://localhost:3000 para acessar. Será pedido o login. Por padrão é gerado um usuário admin: username: admin senha: admin. Nas próximas telas apenas clique na opção `Continue` e `Go to your workspace`.

Para configurar o Bot no rocketchat e conseguir conversar com ele pelo próprio rocket, acesse o [link](http://github.com/lappis-unb/rasa-ptbr-boilerplate/tree/master/docs/add_bot_rocketchat.md).


Opcionalmente, é possível fazer uma configuração para que o assistente virtual inicie a conversa. Para isso você deve criar um `trigger`.
Para criar um `trigger` entre no rocketchat como `admin`, no painel do Livechat na
seção de Triggers, clique em `New Trigger`, preenchendo o Trigger da seguinte forma:


```yaml
Enabled: Yes
Name: Start Talk
Description: Start Talk
Condition: Visitor time on site
    Value: 3
Action: Send Message
 Value: Impersonate next agent from queue
 Value: Olá!
```


#### Instalação

Para executar o Bot em um site, você precisa inserir o seguinte código JAVASCRIPT na sua página:

```js
<!-- Start of Rocket.Chat Livechat Script -->
<script type="text/javascript">
// !!! Mudar para o seu host AQUI !!!
host = 'http://localhost:3000';
// !!! ^^^^^^^^^^^^^^^^^^^^^^^^^^ !!!
(function(w, d, s, u) {
    w.ROCKETCHAT = function(c) { w.ROCKETCHAT._.push(c) }; w.ROCKETCHAT._ = []; w.ROCKETCHAT.url = u;
    var h = d.getElementsByTagName(s)[0], j = d.createElement(s);
    j.async = true; j.src = host + '/packages/rocketchat_livechat/assets/rocketchat-livechat.min.js?_=201702160944';
    h.parentNode.insertBefore(j, h);
})(window, document, 'script', host + '/livechat');
</script>
<!-- End of Rocket.Chat Livechat Script -->
```

**Atenção**: É preciso alterar a variável `host` dentro do código acima para a url do site onde estará o seu ROCKETCHAT.

### TELEGRAM

Após realizar o [tutorial](/docs/setup_telegram.md) de exportação de todas variáveis de ambiente necessárias, é possível realizar a execução do Bot no TELEGRAM corretamente.

<strong><em>Antes de seguir adiante. Importante:</strong></em> As variáveis de ambiente são necessárias para o correto funcionamento do Bot, por isso não esqueça de exportá-las.

Se ainda não tiver treinado seu Bot execute antes:

```sh
make train
```
**Atenção**: o comando "make train" executa um container docker, caso precise de sudo em seu computador
para execução docker, utilize "sudo make train".  


Depois execute o Bot no TELEGRAM:

```sh
sudo docker-compose up bot_telegram
```

### Console

```sh
make train
sudo docker-compose run --rm Bot make run-console
```

### Train Online

```
make train
sudo docker-compose run --rm coach make train-online
```

## Analytics

Para a visualização dos dados da interação entre o usuário e o ChatBot, é utilizado parte da Stack do ELASTIC, composta pelo ELASTICSEARCH e o KIBANA. Com isso, utilizamos um broker para fazer a gerência das mensagens. É possível adicionar mensagens ao ELASTICSEARCH independente do tipo de mensageiro utilizado.

#### Setup RabbitMQ

Primeiramente para fazer o setup do analytics é necessário subir o RabiitMQ e suas configurações.

Inicie o serviço do servidor do RabbitMQ:

```sh
sudo docker-compose up -d rabbitmq
```

Inicie o serviço do consumidor do RabbitMQ, que ficará responsável por enviar as mensagens para o ELASTICSEARCH:

```sh
sudo docker-compose up -d rabbitmq-consumer
```

Lembre-se de configurar as seguintes variáveis de ambiente do serviço `rabbitmq-consumer` no `docker-compose`.

```sh
ENVIRONMENT_NAME=localhost
BOT_VERSION=last-commit-hash
RABBITMQ_DEFAULT_USER=admin
RABBITMQ_DEFAULT_PASS=admin
```

As configurações de `RABBITMQ_DEFAULT_USER` e `RABBITMQ_DEFAULT_PASS` devem ser as mesmas definidas no serviço do `rabbitmq`.


### Execução

Existem duas formas para executar o Asha com o *broker*. A primeira delas é via linha de comando.
Para utilizar esta forma é preciso definir Dentro do arquivo `endpoints.yml` as configurações do broker:

```yml
event_broker:
  url: rabbitmq
  username: admin
  password: admin
  queue: bot_messages
```

Depois basta executar o Bot:

```sh
sudo docker-compose run --rm Bot make run-console-broker
```

A segunda forma é utilizando o script `run-rocketchat` que é utilizado quando o Bot é executado com o ROCKETCHAT como canal. Para isso, as mesmas variáveis devem ser configuradas no arquivo `docker Bot.env`.
Lembre-se também de configurar como `True` a seguinte variável do serviço  Bot` no arquivo `docker Bot-rocketchat.env`.

```
# Broker config
BROKER_URL=rabbitmq
BROKER_USERNAME=admin
BROKER_PASSWORD=admin
QUEUE_NAME=bot_messages
```

Ao final é necessário buildar novamente o container do Bot.

```
sudo docker-compose up --build -d Bot
```

### Setup ELASTICSEARCH

O ELASTICSEARCH é o serviço responsável por armazenar os dados provenientes da interação entre o usuário e o ChatBot.

As mensagens são inseridas no índice do ELASTICSEARCH utilizando o *broker* RabbitMQ.

Para subir o ambiente do ELASTICSEARCH rode os seguintes comandos:

```
sudo docker-compose up -d elasticsearch
sudo docker-compose run --rm -v $PWD/analytics:/analytics Bot python /analytics/setup_elastic.py
```

Lembre-se de setar as seguintes variaveis de ambiente no `docker-compose`.

```
ENVIRONMENT_NAME=localhost
BOT_VERSION=last-commit-hash
```

### Setup KIBANA (Visualização)

Para analisar os dados das conversas com o usuário, utilizamos o KIBANA para ver como os usuários estão interagindo com o Bot, os principais assuntos, média de usuários e outras informações da análise de dados.

O KIBANA nos auxilia com uma interface para criação de visualização para os dados armazenados nos índices do ELASTICSEARCH.

```
sudo docker-compose up -d KIBANA
```

**Atenção:** Caso queira configurar permissões diferentes de usuários (Login) no ELASTICSEARCH/KIBANA, siga esse tutorial ([link](https://github.com/lappis-unb/rasa-ptbr-boilerplate/tree/master/docs/setup_user_elasticsearch.md)).

#### Importação de Dashboards

Para subir com os dashboards criados para fazer o monitoramento de bots:

```
sudo docker-compose run --rm KIBANA python3.6 import_dashboards.py
```

Após rodar o comando anterior os dashboards importados estarão presentes no menu management/KIBANA/Saved Objects. Você pode acessar o KIBANA no `locahost:5601`



## Testando Fluxos de Conversa

É possível testar os fluxos de conversação utilizando o [Evaluation do RASA CORE](https://github.com/lappis-unb/tais/wiki/Testes-Automatizados). Para testá-los no seu Bot basta adicionar um arquivo dentro do diretório  Bot/e2e/` com as histórias a serem testadas. Essas histórias devem ser descritas normalmente, porém com exemplos de frases para cada uma das *Intents* sendo testadas, segundo o formato abaixo:

```
## História de teste 1
* cumprimentar: oi
   - utter_cumprimentar
* action_test: test custom action
   - action_test
```

Uma vez que os arquivos de teste foram adicionados ao diretório correto, basta rodar os testes com a *task* do Bot:

```sh
sudo docker-compose run --rm Bot make test-stories
```

Para gerar data-science referente aos testes automatizados do Bot, execute o seguinte comando do *Makefile* na raíz do projeto:

```sh
sudo docker-compose run --rm Bot make test-dialogue
```

## Notebooks - Análise de Dados

### Setup

Levante o container `notebooks`

```sh
docker-compose up -d notebooks
```

Acesse o notebook em `localhost:8888`



## Tutorial para levantar toda a stack

```sh
sudo docker-compose up -d rocketchat

sudo docker-compose up -d rabbitmq
sudo docker-compose up -d rabbitmq-consumer

sudo docker-compose up -d elasticsearch
sudo docker-compose run --rm -v $PWD/analytics:/analytics Bot python /analytics/setup_elastic.py
sudo docker-compose up -d KIBANA

sudo docker-compose up -d Bot
```


# Como conseguir ajuda

Em caso de dúvidas em relação ao RASA, veja o grupo [TELEGRAM RASA Stack Brasil](https://t.me/RasaBrasil), a comunidade esta em crescimento no Brasil ou consulte a documentação do RASA [RASA](https://rasa.com/docs/).


# Licença

Todo o framework do boilerplate é desenvolvido sob a licença
https://github.com/marciosborba/asha/blob/master/LICENSE
