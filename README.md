## Configurando o Ambiente MongoDB

1. Execute os contêineres do MongoDB e MongoDB Express: 

```bash
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
