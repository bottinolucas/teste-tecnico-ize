# Teste técnico - IZE Gestão Empresarial

Este repositório é dedicado para a resolução da fase 2 do processo seletivo de Engenheiro(a) de Dados Júnior da IZE Gestão Empresarial. Esta etapa consiste em um pequejo projeto prático que envolve a resolução de dois desafios.

A empresa busca um Engenheiro de Dados para ajudar a otimizar a logística de entrega de produtos. O desafio foi preparado para avaliar a capacidade do candidato de resolver problemas e projetar soluções de dados.

## Desafio 1 

**CONTEXTO**: A empresa utiliza diferentes sistemas para registrar informações sobre envios e entregas. O desafio envolve o processamento de dados de um arquivo CSV que contém informações de rastreamento de pacotes. 

-   Arquivo recebido: rastreamento.csv
    - Colunas: id_pacote, origem, destino, status_rastreamento, data_atualizacao 

Perguntas sobre o desafio 1 

1.  Descreva uma pipeline de ingestão de dados para processar esse arquivo CSV. Qual ferramenta ou tecnologia você usaria em cada etapa (ingestão, processamento, armazenamento)?
2.  Proponha um modelo de dados para armazenar as informações de rastreamento de forma eficiente em um banco de dados relacional (SQL). Considere a possibilidade de um pacote ter múltiplos eventos de rastreamento ao longo do tempo (Ex: "EM TRANSITO", "EM ROTA", "ENTREGUE")
3.  Descreva a abordagem escolhida e justifique a escolha das tecnologias e o design do modelo de dados. 

## Desafio 2 

**CONTEXTO**: A equipe de análise de negócios precisa monitorar a eficiência das entregas para identificar gargalos e otimizar rotas. Para isso, precisamos de um sistema de relatórios que consuma os dados de rastreamento.

Perguntas sobre o desafio 2

1.  Proponha uma solução para criar um dashboard em tempo quase real que mostre o número de pacotes por status (EM TRÂNSITO, ENTREGUE, etc) e a média de tempo de entrega (do envio até a entrega final).
2.  Quais seriam as implicações se, no futuro, a empresa passar a receber dados de rastreament oem tempo real via um endpoint de API, em vez de um arquivo CSV diário? O que você precisa adaptar no seu pipeline e modelo de dados?

## SOLUÇÃO DO DESAFIO  1

A pipeline de ingestão de dados utilizada para resolver o desafio 1 é baseada em um processo de ingestão por lotes, onde o arquivo `rastreamento.csv` é a fonte de dados utilizada e o banco de dados SQL Postgres é o destino final. 

A arquitetura utilizada para construir essa pipeline foi a arquitetura de medalhão, bastante comum na construção de data warehouses, baseando-se nos estágios bronze, prata e ouro: 

![Pipeline de Ingestão de Dados](/assets/pipeline-ingestao-dados.png)

1.  Camada Bronze (DADOS BRUTOS)
    
    A camada bronze é o ponto de partida da pipeline, recebendo os dados da fonte original sem qualquer alteração.

    Arquivos presentes:

        rastreamento.csv

2.  Camada Silver/Prata (Limpeza e Transformação)

    A camada silver/prata é a etapa onde os dados presentes na camada bronze são limpos, padronizados e transformados para se adequarem ao modelo relacional de dados.

    Arquivos presentes:

        etl_bronze_to_silver.ipynb
        rastreamento_tratado.csv

3.  Camada Gold/Ouro (Modelagem de dados e armazenamento final)

    Por fim, a camada Gold/Ouro é a camada responsável por aplicar a regra de negócio, garantindo que os dados sejam armazenados de forma estruturada no banco de dados.

    Arquivos presentes:

        etl_silver_to_gold.ipynb

## SOLUÇÃO DO DESAFIO 2

Para a construção de um sistema de relatórios que consuma os dados de rastreamento, o ideal é que a modelagem do sistema esteja implementada como um Data Warehouse, ideal para construção de relatórios e BI. Para que o monitoramento seja eficiente e possua velocidade na hora de fazer relatórios, torna-se necessário uma modelagem menos estruturada e que facilite todo o processo de análise de dados, com o star scheme se tornando a modelagem ideal para este problema.

A mudança do consumo de dados para um endpoint de API em tempo real é fundamental para a arquitetura do sistema construído, pois ao invés de consumir os dados em lotes, estariamos consumindo dados continuamente