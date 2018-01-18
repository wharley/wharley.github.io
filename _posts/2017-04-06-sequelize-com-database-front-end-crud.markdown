---
layout: post
title: Criando um CRUD com Angular e Nodejs + Sequelize
date: 2017-04-06 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post-3.jpg # Add image post (optional)
tags: [Blog, Angular]
author: # Add name author (optional)
---
Para ficar completo o post anterior [(ORM Sequelize com database existente do Postgres no Nodejs)](2017-03-24-sequelize-com-database-existente-postgres.markdown), criei um crud para melhor visualização das informações.

Usei o framework [Angular][angular]{:target="_blank"}, que permite desenvolver aplicações web e mobile mantido pela Google.

Apesar de ter mantido o nome Angular é outro conceito de desenvolvimento, sendo assim quem quiser aprender sugiro a nova versão.

# Verificações de instalações!

Antes de começar é necessário certificar que o [Nodejs 6.9.5][node]{:target="_blank"} (ou Superior) e o [NPM 3][npm]{:target="_blank"} (ou Superior) estão instalados.

{% highlight bash %}
$ node -v
$ npm -v
{% endhighlight %}

Instalei uma das maiores novidades do **Angular** o [@angular/cli][angularcli]{:target="_blank"} (para aplicações angulares baseadas no projeto ember-cli) e a linguagem adotada é o [Typescript][typescript]{:target="_blank"}, para desenvolvimento **JavaScript** em larga escala. Com **TypeScript** é possível escrever código utilizando uma estrutura fortemente tipada e ter este código compilado para **JavaScript** puro. Qualquer navegador, qualquer host.

{% highlight bash %}
$ npm i -g typescript @angular/cli
{% endhighlight %}

Criei o projeto **Angular** dentro do diretório **database_existente** (projeto do post anterior)

{% highlight bash %}
$ ng new myApp
$ cd myApp
{% endhighlight %}

Testei o projeto utilizando o comando abaixo

{% highlight bash %}
$ ng serve
{% endhighlight %}

Gerando e atendendo um projeto Angular através de um servidor de desenvolvimento navegue até **_http://localhost:4200/_**. O aplicativo será recarregado automaticamente se você alterar qualquer um dos arquivos de origem.

Pode-se configurar o host e a porta **HTTP** padrão usados ​​pelo servidor de desenvolvimento com duas opções de linha de comando:

{% highlight bash %}
$ ng serve --host 0.0.0.0 --port 4201
{% endhighlight %}
![ngserve]({{site.baseurl}}/assets/img/ngserve.jpg)
_Pronto, projeto criado com sucesso._

_Deixei o **ng serve** executando, assim toda alteração feita irá compilar e mostrar no navegador_

Utilizei o framework [Bootstrap][bootstrap]{:target="_blank"}, [jquery][jquerys]{:target="_blank"} e o [fontawesome][fonts]{:target="_blank"}

{% highlight bash %}
$ npm i --save bootstrap jquery font-awesome
{% endhighlight %}

Alterei o `.angular-cli.json` para adicionar o caminho dos mesmos

{% highlight json %}
//Antes
        "styles": [
            "styles.css"
        ],
        "scripts": []
//Depois adicionei o bootstrap, jquery e o font-awesome
        "styles": [
            "../node_modules/bootstrap/dist/css/bootstrap.css",
            "../node_modules/font-awesome/css/font-awesome.css",
            "styles.css"
        ],
        "scripts": [
            "../node_modules/jquery/dist/jquery.js",
            "../node_modules/bootstrap/dist/js/bootstrap.js"
        ]
{% endhighlight %}

# Construindo o Front-end

Adicinei um [navbar][navbars]{:target="_blank"} no arquivo `myApp/src/app/app.component.html`

{% highlight html %}
<nav class="navbar navbar-default">
    <div class="container-fluid">
    <!-- Brand and toggle get grouped for better mobile display -->        
        <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
            <a class="navbar-brand" href="#">Menu</a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav">
                <li class="dropdown">
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Cadastro <span class="caret"></span></a>
                    <ul class="dropdown-menu">
                        <li><a href="#">Setor</a></li>
                        <li><a href="#">Produto</a></li>
                    </ul>
                </li>
            </ul>
        </div>
        <!-- /.navbar-collapse -->
    </div>
    <!-- /.container-fluid -->
</nav>
{% endhighlight %}

Verifiquei se o `ng serve` estava rodando.

![navbar]({{site.baseurl}}/assets/img/navbar.jpg)

Criei o **Component** (Representam uma unidade unificada de **_​View + Model + Style_** que você deve compor para criar uma aplicação) dentro do diretório `myApp/src/app/`

{% highlight bash %}
$ ng g c setor
{% endhighlight %}

_Generate / Component_ - gerou-se os arquivos `setor.component.css`, `setor.component.html`, `setor.component.spec.ts`, `setor.component.ts` e importei no `app.module.ts`

Criei um arquivo `setor.service.ts` dentro do diretório _src/app/setor_

[npm]: https://www.npmjs.com/get-npm?utm_source=house&utm_medium=homepage&utm_campaign=free%20orgs&utm_term=Install%20npm
[node]: https://nodejs.org/en/
[angular]: https://angular.io/
[angularcli]: https://github.com/angular/angular-cli#installatio
[typescript]: https://www.typescriptlang.org/docs/tutorial.html
[bootstrap]: http://getbootstrap.com/
[jquerys]: https://jquery.com/
[fonts]: http://fontawesome.io/
[navbars]: https://getbootstrap.com/docs/3.3/components/#navbar
