## Propósito desse projeto 

Este projeto tem como objetivo analisar o comportamento das transações PIX ao longo do tempo, buscando compreender padrões de uso, volume financeiro, frequência de transações e possíveis variações temporais. A partir dessa análise, é possível obter uma visão mais clara sobre a dinâmica do PIX como meio de pagamento amplamente utilizado no Brasil.

O MongoDB foi escolhido como banco de dados por sua flexibilidade no armazenamento de grandes volumes de dados semi-estruturados, além de sua boa performance para consultas analíticas em datasets extensos, característica essencial para um contexto de Big Data.

Já o Metabase foi utilizado como ferramenta de visualização por permitir a criação rápida e intuitiva de dashboards interativos, facilitando a exploração dos dados, a construção de análises temporais e a comunicação dos resultados de forma visual e acessível.

Dessa forma, a combinação entre MongoDB e Metabase possibilita uma análise eficiente, escalável e visualmente rica das transações PIX, apoiando a interpretação do seu comportamento ao longo do tempo.


## Configurando o Ambiente MongoDB

1. Execute os contêineres do MongoDB e MongoDB Express: 

```bash
chmod +x permissions.sh && ./permissions.sh
docker-compose build
docker-compose up -d
```

2. Verifique se os contêineres estão ativos e sem erros de implantação: 

```bash
docker ps
docker-compose logs
```
## Acesso GUI: MongoDB Express 

1. Acesse o MongoDB Express pelo navegador (`http:\\localhost:8081`) e forneça as credenciais: 

- **Usuário**: admin
- **Senha**: pass

2. Clique em `Create Database` e crie uma base de dados . Dentro da base de dados, clique em `Create Collection` e crie a uma coleção.
    
   2.1 OBS: dentro da pasta `datasets` do `mongodb` eu já adicionei o arquivo/dataset lá, não sendo assim necessário o download do arquivo na sua máquina.

## Acesso CLI: MongoDB 

1. Acesse o Shell do Contêiner MongoDB: 
```bash
docker exec -it mongo_service /bin/bash
```
2. Autentique-se no MongoDB: 
```bash
mongo -u root -p mongo
```

3. Importe o dataset: 
```bash
mongoimport \
  --db <nome_database> \
  --collection <nome_collection> \
  --type csv \
  --headerline \
  --file datasets/Estatisticas_de_transações_pix.csv
```
4. Confira se o import deu certo (por volta de 270.000 documentos deveriam ser importado)

5. Rode isso aqui para transformar o campo VALOR que está com o texto(string) para número, substituindo a vírgula por ponto (certifique-se de estar usando o database que você criou):
```bash
db.<nome_database>.updateMany(
  {},
  [
    {
      $set: {
        VALOR: {
          $cond: {
            if: { $eq: [{ $type: "$VALOR" }, "string"] },
            then: {
              $toDouble: {
                $replaceOne: {
                  input: "$VALOR",
                  find: ",",
                  replacement: "."
                }
              }
            },
            else: "$VALOR"
          }
        }
      }
    }
  ]
);

```
7. Valide se não há mais strings no dataset:
```bash
    db.<nome_database>.countDocuments({ VALOR: { $type: "string" } })
```

tudo certo com o MONGO!

# Metabase
## 1. Introdução

1. Execute os contêineres do Metabase:

```bash
chmod +x permissions.sh && ./permissions.sh
docker-compose build
docker-compose up -d
```

2. Acesse a GUI do Metabase pelo navegador (`http:\\localhost:3000`) e clique em `adicionar um banco de dados` , quando abrir a janela adicione as informações assim:
```bash
host : mongo_service
Nome do banco de dados : <database_name_do_mongodb>
Usuário: root
Senha: mongo
```
2.1 Clique em Salvar e pronto ! A partir dai você pode começar a criar seu dashboard e as perguntas.

3. Segue o exemplo que fiz:
   https://github.com/Paulohspaniago/big_data_pix_transactions/blob/main/docs/pdf/dashboard1.pdf
   https://github.com/Paulohspaniago/big_data_pix_transactions/blob/main/docs/pdf/dashboard2.pdf
