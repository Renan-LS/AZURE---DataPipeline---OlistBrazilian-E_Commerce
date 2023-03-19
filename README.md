# AZURE---DataPipeline---OlistBrazilian-E_Commerce #
## Escopo do projeto ##
Este projeto visa a cronstrução de um pipeline completo para automação das etapas de ETL, utilizando das mais diversas funções disponibilizadas pela cloud AZURE.

## Base de dados ##
A Olist é uma empresa brasileira de tecnologia que oferece soluções para e-commerce e marketplace, ajudando pequenos e médios empreendedores a venderem seus produtos online. Sua plataforma permite que os empreendedores criem uma loja virtual integrada a um marketplace, ampliando as possibilidades de venda. Além disso, a empresa oferece uma solução completa de logística e suporte ao cliente para seus parceiros.
A base de dados escolhida foi a da OLIST, disponível no kaggle.
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce?resource=download

<center>Segue descrição da arquitetura da base de dados. </center>

![image](https://user-images.githubusercontent.com/120025497/226148405-da9eb15a-7c8f-48ea-9193-d4fe3465e550.png)

## Arquitetura do projeto ##
O projeto utiliza a arquitetura em nuvem da Microsoft, <b>Azure</b>, que é baseada em serviços em nuvem. O Azure oferece recursos e ferramentas para ajudar no desenvolvimento e implantação de aplicativos, além de garantir alta disponibilidade, segurança e desempenho.

![image](https://user-images.githubusercontent.com/120025497/226148620-60539001-41e1-488f-8de7-55f5684eaa8a.png)

Alguns dos serviços utilizados são:
  - Azure Data Factory
  - Azure Key Vault
  - Azure Storage Account(Data Lakes)
  - Azure Databricks Workspace
  - Azure Synapse Workspace
  - SQL Server
  - SQL Database
  - Azure Active Directory
  
Além do gerenciamento de assinaturas, permissões, regiões, urls, tokens, linked services, clusters, pipelines, triggers, pools, cost estimations, dentre outros.

## Descrição do projeto ## 

- Nesta seção irei destacar alguns passos e desafios enfrentados ao decorrer deste projeto.
O primeiro passo do projeto foi a importação da base de dados do keagle(link na Seção Base de Dados).
![image](https://user-images.githubusercontent.com/120025497/226148984-2429edc2-4115-4244-a911-8de2e6de23f3.png)

- Feito isto, e após ser criada uma conta na plataforma da Azure, a primeira coisa a ser feita é a criação de um grupo de recursos, que nada mais é que um contêiner lógico que agrupa recursos(meio óbvio, não!?) relacionados a um projeto ou solução. Essa prática traz diversos benefícios, como a possibilidade de gerenciar e monitorar vários recursos ao mesmo tempo e a facilidade na implantação e remoção de recursos.
Em seguida foi criado um servidor SQL de modo a simular uma situação real em que os CSV's estariam disponiveis para EXTRAÇÃO (o E de E-T-L), é neste local que que os dados extraídos do KAGGLE(Olist) serão armazenados!!

- Uma vez que a base de dados está preparada, agora é hora de começar a subir os recursos necessários para o nosso Data Lake, mas o que é isso? 
Data Lake é um repositório centralizado de dados em grande escala, que armazena dados brutos de várias fontes em seu formato original. O objetivo é possibilitar o acesso rápido e fácil a grandes volumes de dados para análise e tomada de decisão, sem a necessidade de transformá-los em um formato estruturado e organizado. 
E então é criada uma **conta de armazenamento(Storage Account)** para hospedar nosso Data Lake, e dentro dela, criam-se os containeres landing, processing e curated para armazenamento dos dados de maneira organizada.

![image](https://user-images.githubusercontent.com/120025497/226149492-b34d71d8-2e3d-4487-8414-ee93fdcd778f.png)

- Data Lake OK, Base de dados OK.. Chega a hora de iniciar a construção do pipeline! Mas afinal, o que é Pipeline de dados?
- É um conjunto de processos automatizados que permitem a transferência de dados de uma ou mais fontes para um ou mais destinos!! Resumindo: É o CORE do nosso projeto!
- E para tal tarefa, o AZURE disponibiliza uma ferramente INCRÍVEL chamada **Data Factory**, responsável por orquestrar todo fluxo de dados! A primeira tarefa a ser realizada por este pipeline
é a Extração dos dados do SQL Server para a LANDING de nosso Data Lake. A imagem que segue ilustra a configuração do ForEach que permite que o Data Factory itere sobre uma lista de itens, no nosso caso os CSV's, e realize uma determinada operação
,no caso copiar de um local, para o outro.

![2 -pipeline   for each todos de uma vez](https://user-images.githubusercontent.com/120025497/226150021-f2afeca9-7050-4abe-9e24-7f09aff4c8ac.jpg)

- Após a execução da parte de Extração do pipeline, é verificado na LANDING que de fato a operação foi realizada com sucesso!
![4 - apos execucacao do pipeline_verificado o povoamento de nossa landing](https://user-images.githubusercontent.com/120025497/226150161-7d7443fd-a373-4a66-ac26-ca20332ef508.jpg)

- O próximo passo é iniciar a realização da fase de PROCESSING. E para isto, é criado um workspace no DATABRICKS. Mas afinal o que é isto? 
- O DATABRICKS é uma plataforma de análise de dados poderosa e flexível que oferece recursos avançados para processamento de dados MASSIVO, de maneira distribuída, o que o torna altamente escalável e elástico.
![6 - criando cluster no databriks, o mais basico possivel](https://user-images.githubusercontent.com/120025497/226150379-ac34a39f-9fed-4a4c-97a3-99ca9229d3af.jpg)
Na imagem acima é criada um cluster no Databricks! Ps: com as configurações mais básicas possíveis(e mais baratas!!), pois a carga de processamento de nosso trabalho não requer maiores configurações.
Ps: Tive um certo problema ao tentar realizar a criação do cluster a priori, mas depois de várias tentativas frustradas descobri que o problema estava na região que eu havia criado o workspace do databricks(CENTRAL US). Havia escolhido
esta devido ao custo ser menor. Porém devido ao erro, tive de escolher outra região mais cara... não teve jeito!

- Nesta etapa são realizados algumas operações utilizando de PySpark como transformação dos dados para formato PARQUET,criação de um servidor SQL para disponibilização dos dados através de tabelas e databases.(O que facilita o trabalho para quem se utiliza de soluções para DataViz como Power BI)

