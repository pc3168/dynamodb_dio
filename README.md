# Projeto Utilizando Banco de dados NoSql Dynamodb

## Utilização do dynamoDB Localmente

### Windows


[link para Download DynamoDB] (https://s3.sa-east-1.amazonaws.com/dynamodb-local-sao-paulo/dynamodb_local_latest.zip)

[link para Donwlaod do AWS CLI] (https://awscli.amazonaws.com/AWSCLIV2-2.0.30.msi)

### Linux - AWS CLI 
### para realizar o donwload no Ubuntu utiliza o comando abaixo.
```
sudo apt-get install awscli
```

## Depois utilizei o comando (windows / linux)

```
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb
```

## para configurar as credenciais e habilitar a autorização

```
aws configure
```

```
AWS Access Key ID: "fakeMyKeyId"
AWS Secret Access Key: "fakeSecretAccessKey"
Região eu digitei: Brasil
```

## e por fim utilizei o comando para verificar se estava tudo ok.

```
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

### link para documentação.

[Link da Documentação DynamoDB Local] (https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html#DynamoDBLocal.DownloadingAndRunning.title)

[LInk da documentação AWS CLI] (https://docs.aws.amazon.com/pt_br/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)



## Serviço utilizado

- Amazon dynamoDB Local 
- Amazon CLI para execução em linha de comando. 

## Comando para execução do experimento:

- Criar uma tabela (Como utilizei no Windows tive que colocar tudo em uma linha.)

```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5 --endpoint-url http://localhost:8000
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Music \
    --item file://src/itemmusic.json  --endpoint-url http://localhost:8000
```

- Inserir múltiplos itens
```
aws dynamodb batch-write-item \
    --request-items file://src/batchmusic.json --endpoint-url http://localhost:8000
```

- Consultando os itens da tabela Music
```
aws dynamodb scan --table-name Music --endpoint-url http://localhost:8000
```

- Criar um index global secundário baeado no título do álbum  
```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" --endpoint-url http://localhost:8000
```


- Criar um index global secundário baseado no nome do artista e no título do álbum
```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" --endpoint-url http://localhost:8000
```

- Criar um index global secundário baseado no título da música e no ano
```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=SongYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" --endpoint-url http://localhost:8000
```

- Pesquisar item por artista
```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Iron Maiden"}}' --endpoint-url http://localhost:8000
```

- Pesquisar item por artista e título da música
```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist and SongTitle = :title" \
    --expression-attribute-values file://src/keyconditions.json --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado no título do álbum
```
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Fear of the Dark"}}' --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado no nome do artista e no título do álbum
```
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Iron Maiden"},":v_title":{"S":"Fear of the Dark"} }' --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado no título da música e no ano
```
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Wasting Love"},":v_year":{"S":"1992"} }' --endpoint-url http://localhost:8000
```

- Comando para consultar todos os registros
```
aws dynamodb scan --table-name Music  --endpoint-url http://localhost:8000
```

- Filtrar por um atributo específico:
```
aws dynamodb scan --table-name Music --filter-expression "artist = :a" --expression-attribute-values '{":a":{"S":"Michael Jackson"}}'  --endpoint-url http://localhost:8000
```

- Limitar a quantidade de resultados
```
aws dynamodb scan --table-name Music --limit 2  --endpoint-url http://localhost:8000
```

- Criando uma tabela chamada produtos com uma chave primária "id" do tipo string e definindo uma capacidade de leitura e escrita de 5 unidades:
```
aws dynamodb create-table --table-name produtos --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 --endpoint-url http://localhost:8000
```

- Adicionando um registro na tabela produtos
```
aws dynamodb put-item --table-name produtos --item '{"id": {"S": "1"}, "nome": {"S": "Produto A"}, "preco": {"N": "10.99"}}' --endpoint-url http://localhost:8000
```

- Consultando todos os registros da tabela produtos
```
aws dynamodb scan --table-name produtos --endpoint-url http://localhost:8000
```

- Consultando apenas o produto com o "id" igual a "1"
```
aws dynamodb get-item --table-name produtos --key '{"id": {"S": "1"}}' --endpoint-url http://localhost:8000
```