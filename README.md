# DESAFIO DE DOCKER
Exercício para aplicar conhecimentos introdutórios de Docker

## OBJETIVO GERAL
O objetivo desse desafio é colocar em prática os conhecimentos introdutórios de Docker vistos em nosso curso introdutório, trabalhando com um cenário real e bem interessante: monitoramento de um container com uma instância do MongoDB!

## FERRAMENTAS
Quando lidamos com aplicações rodando em ambientes de produção, é necessário analisarmos como as aplicações (e as máquinas em que estão rodando) estão trabalhando: saber se um banco de dados está realizando muitas leituras em disco, saber se a rede está ficando sobrecarregada, saber se alguma aplicação está usando mais memória do que devia, etc. Duas tecnologias que são muito utilizadas para esta tarefa são:

* [Prometheus](https://prometheus.io/), uma ferramenta open source para armazenamento e consulta de séries temporais (medidas coletadas com o passar do tempo);
* [Grafana](https://grafana.com/), uma ferramenta para visualização e monitoramento de dados de séries temporais;

Uma arquitetura comum quando utilizamos essas ferramentas é a seguinte:

![Diagrama Prometheus/Grafana](https://github.com/InsightLab/docker-introduction-challenge/raw/master/Diagrama%20Prometheus_Grafana.png)

O Grafana entra na parte de visualização, com seus painéis (*dashboards*). Eles são configurados com consultas e gráficos para apresentar os dados do Prometheus. Este último, por sua vez, armazena as métricas coletadas das aplicações e responde às consultas do Grafana. Por fim, existe o **exporter**, o qual é responsável por coletar as métricas de uma aplicação e disponibilizá-las ao Prometheus. O *exporter* pode aparecer, pelo menos, de três maneiras (da esquerda para a direita): encapsulando uma aplicação, rodando dentro da aplicação ou analisando externamente a aplicação.

Para este desafio, utilizaremos o seguinte ecossistema: haverá uma instância do MongoDB rodando. O Prometheus será responsável por colher métricas dessa instância (número de inserções, consultas, memória utilizada, etc.) e o Grafana irá conter o dashboard para visualizar as métricas do Prometheus. Porém, o Prometheus não se comunica diretamente com o MongoDB, ele precisa coletar as métricas já estruturadas em um formato que ele seja capaz de compreender. Para isso, precisamos usar um exporter: um serviço responsável por analisar a instância do MongoDB e prover as métricas para o Prometheus.

## TUTORIAL

Esse repositório contém um arquivo *docker-compose.yml* que traz as configurações necessárias para subir o Grafana e o Prometheus.

Porém, antes de subir os serviços, é necessário que seja gerada a imagem do exporter do mongoDB para coletar as métricas que serão enviadas para o Prometheus. Para isso, basta clonar o repositório do [exporter do MongoDB para o Prometheus](https://github.com/percona/mongodb_exporter) disponível no github dos desenvolvedores do *Perconalabs*, após isto basta construir uma imagem docker a utilizando a diretiva: `make docker`.

Após a execução, será criada a imagem (*mongodb-exporter:master*) e bastará executar o comando abaixo para subir os serviços. 

```
docker-compose up -d
```

Assim, será possível acessar a porta **9090** que conterá a interface web do Prometheus. Acessando o cabeçalho da página, vá em **status -> targets** onde será mostrada as fontes de onde o Prometheus estará coletando suas métricas. Ele apresentará que está consumindo métricas de um serviço chamado **mongodb-exporter** na porta **9216**, o state do serviço estando **UP**, significa que está tudo OK.

Acessando a porta **9216** você verá como que o exporter envia as métricas.

O Grafana estará disponível na porta **3000**, para acessa-lo, utilize o usuário **admin** e senha **admin**. Após o login, é possível identificar um símbolo em forma de + no painel esquerdo. Ao clicar nele, basta selecionar a opção **Import** e escolher o dashboard disponível no repositório (o arquivo chamado **MongoDB-Dashboard.json**) para que seja criado o dashboard no Grafana.

Para realizar iterações como o mongoDB utilize o Mongo Express, disponível na porta **8081**.

Pronto! Agora conseguimos monitorar nossa instância do MongoDB!

