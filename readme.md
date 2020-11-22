# Sobre o projeto

Criar um layout no blade.php usando o tailwindcss

# Material de Apoio
[LINK DA AULA](https://www.youtube.com/watch?v=oVRWQJE5a1c&t=2583s)

# Sobre o código
 * Laravel 8
 * nginx:1.17.10-alpine
 * php:7.4-fpm
 * redis:alpine
 * phpmyadmin:latest
 * Mysql 8.0
 * [Node 14 LTS](https://github.com/nodesource/distributions#debmanual)

<p>A versão <strong>alpine</strong> por ela mais leve em questão de tamanho de arquivo. Contudo, caso queria usar
a versão mais completa é só tirar o <strong>alpine</strong>. Caso queira atualizar para versão mais recente, apenas acompanha no site
docker hub do php, nginx e entre outros e só visualizar as nomenclatura da versão nova, por exemplo, se tiver atualização, php:7.5, só 
trocar o 7.4 pelo 7.5 que ele vai atualizar, mesmo formato para nginx.</p>
<br>
Uma observação, o meu arquivo <b>Dockerfile-app-php-fpm</b> na <b>linha 12</b> ele dá erro as vezes quando atualiza o laravel, por conta  e umas da dependência precisa do git para, e como não 
tem o git instalado na minha configuração do docker-compose, por isso, tem que executar o comando.

---

````
composer update
````
ou
````
composer install
````

Para instalar as dependências do seu projeto na sua máquina antes de fazer 
deploy para container no docker.

# Ordem de comando

Execute o comando para gerar o build da imagem
```
docker-compose up -d --build
```
Caso esteja com problemas de salvar os arquivos do banco de dados, use esse comando, pode ser
problema de permissão de pasta no seu computador
```
chmod -R 777 dbdata
```

Acessar o container da aplicação se não for versão alpine 
```
docker exec -it nome_container /bin/bash
```
Digite esse comando  
```
cd /var/www/
```
Verifique se o arquivo <strong>.env.example</strong> foi copiado, usando o comando ls -la
Se estiver tudo okay, vai poder  visualizar sua aplicação funcionando.

* PHPMYADMIN -> localhost:8082
* SITE_LARAVEL -> localhost:8081

<p>Precisa está dentro do container da sua aplicação onde ficar os arquivos do laravel na pasta <strong>/var/www/</strong>
para executar os comando do laravel como.

```
php artisan migrate
php artisan make:model algumacoisa
```

<p> São apenas alguns exemplos </p>

# DOCKER-COMPOSE.yaml
Essas são configurações que você editar para conectar ao seu banco de dados <br>
e também para configurar o <strong> Mysql </strong>

## configuração do service app
```
environment:
      - DB_DATABASE=laravel
      - DB_PASSWORD=root
      - DB_USERNAME=root
      - DB_HOST=db
```

## configuração do service mysql
```
  environment:     
      - MYSQL_DATABASE=laravel
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=root
      - MYSQL_PASSWORD=root
```

## Teste para verificar se o banco de dados se está OKAY
```
   healthcheck:
      test: ["CMD-SHELL", 'mysql -uroot --database=laravel --password=root --execute="SELECT count(table_name) > 0 FROM information_schema.tables;" --skip-column-names -B']
      interval: 20s
      timeout: 10s
      retries: 4
```
Código acima vai verificar se o banco de dados está funcional e se está conectando com as suas configurações. O provisamento do banco de dados 
demora mais do que outros containers.

## Configurar  Nginx
#### No arquivo dockerfile do nginx 
```
server {
    listen 80;
    index index.php index.html;
    root /var/www/public;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```
Atenção nessa parte <strong> fastcgi_pass app:9000; </strong> o nome <strong> app </strong> é do service que está no <strong> docker-compose </strong>, se alterar o nome do service no <strong>docker-compose </strong> é preciso alterar aqui também, porque aqui faz a mágica acontecer e parte mais importante, pois é daqui que ele vai redirecionar a sua aplicação para porta 9000

Caso altere alguma coisa nessa parte, execute o comando
```
docker-compose up -d --build
```

# Observação

Após finalizar a criação da imagem do seu container, acesse ele pelo comando
```
docker exec -it nome_container /bin/bash
```
e depois
```
cd /var/www/
```

Verifique se o arquivo <b>.env.example</b> foi copiado, usando o comando 
```
ls -la
```

Se encontrar, beleza, vamos para próximo passo.
Digite  
```
cp .env.example env.
```
Logo após
```
php artisan key:generate
```

# Problema permission denied storage

Se tiver problemas com storage é a questão de permissão. Pode utilizar esse comando no terminal

```
docker exec app chmod -R 777 /var/www/storage
```

#Problema com usuário

Use esse comando no terminal, ele vai pegar usuário do seu computador e adicionar no grupo dos usuários
para editar o arquivo.

```
docker-compose exec php chown -R $USER:www-data /var/www/caminho_do_arquivo_quer_editar_ou_pasta
```

# Na hora de ignorar algumas pastas

Tente ignorar a pasta dbdata para upar para o repositório. Sempre que possível gere um arquivo sql do banco de dados e salva na pasta SQL.

O motivo é porque vai ficar muito grande seu repositório.


## Como ignorar a pasta

[Onde aprendi](https://gist.github.com/kelvinst/7d508da482d13bb301c9)

## Como fazer um `.gitignore` local?

Bom, este é um recurso, como muitos outros, bem escondido do git. Então resolvi fazer um post para explicar a situação em que pode-se usar e como fazer essa magia negra. :ghost:

## O problema

Você provavelmente já adicionou algum dia um arquivo no projeto que não deveria ser commitado certo? E como você fez para ignorar esse arquivo mesmo? Provavelmente adicionou no arquivo `.gitignore`.

OK então, aí você commitou esse arquivo `.gitignore` e pronto, mais ninguém poderá criar um arquivo com o mesmo nome e commitar. Mas espera aí! Não era isso que você queria! Você só queria ignorar esse arquivo na sua máquina, se alguém, algum dia por obséquio achar esse um nome bom para seu arquivo, que assim seja.

Então como fazer isso? Não commitar o arquivo `.gitignore` e colocar o `.gitignore` dentro do `.gitignore` para não commitar ele por quando tiver alteração. Bom, essa opção se você pensar um pouco vai notar porque não funciona: se você disser para o git ignorar o `.gitignore`, como é que você vai commitar o `.gitignore` com o `.gitignore` ignorado (nossa, quanta ignorância :grin:).

OK, como posso fazer então?

## A solução!

Então, aqui vai uma maneira para você fazer isso. Em todo repositório git existe um arquivo `.git/info/exclude`. Ele funciona exatamente como um arquivo `.gitignore` só que ele não é commitado! Então é só colocar uma linha com o nome do seu arquivo nele e :tada:!
