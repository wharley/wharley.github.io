---
layout: post
title:  Criando APIs incríveis usando a library node-restful
date:   2018-02-17 13:32:20 +0300
description: É uma biblioteca para fornecer rapidamente uma API REST com express. Com ela você registra recursos de mongoose e as rotas RESTful padrão são feitas automaticamente. # Add post description (optional)
img: post-8.jpeg # Add image post (optional)
tags: [Blog, Nodejs]
author: Wharley Ornelas # Add name author (optional)
---

> Como de costume escrevo sobre minhas vivências, neste post irei falar sobre uma experiência fantástica que tive em um projeto.

Surgiu um projeto de uma api restful utilizando o mongodb, como tive uma experiência com o framework LoopBack pensei em usá-lo devido o tempo que teria, resolvi da uma pesquisada sobre a existência de algo diferente e que por sinal cheguei em uma library linda :heart: que me chamou muito atenção pela simplicidade e flexibilidade.

# Node-restful

É uma biblioteca para fornecer rapidamente uma `API REST` com express. Com ela você registra recursos de mongoose e as rotas `RESTful` padrão são feitas automaticamente.

{% highlight javascript %}
const express = require('express'),
    restful = require('node-restful'),
    mongoose = restful.mongoose;
const app = express()

const Resource = restful.model('resource', mongoose.Schema({
    title: 'string',
    year: 'number',
  })
  .methods(['get', 'post', 'put', 'delete'])

Resource.register(app, '/resources')

app.listen(3000)
{% endhighlight %}

> A melhor parte é que `restful.model` retorna um modelo de `Mongoose`, para que você possa interagir com ele da forma que você já está acostumado ou seja, `new Resource`, `Resource.findById` etc.

{% highlight javascript %}
GET /resources
GET /resources/:id
POST /resources
PUT /resources/:id
DELETE /resources/:id
{% endhighlight %}

Existem algumas funções disponíveis depois de registrar o esquema de mongoose.

```
.methods([…])
```

> Há uma lista de métodos que devem estar disponíveis no recurso. Caso deseje não autorizar algum método apenas registre as que serão utilizadas.

{% highlight javascript %}
Resource.methods(['get', 'post', 'put'])
{% endhighlight %}

Também podemos executar rotas personalizadas:
{% highlight javascript %}
.route(path, handler)
{% endhighlight %}
{% highlight javascript %}
Resource.route('recommend', (req, res, next) => {
  res.send('I have a recommendation for you!')
})
{% endhighlight %}

Ou fazer combinações de métodos `HTTP`; Suponhamos que seja necessário executar códigos antes ou depois de uma rota, e que seja necessário obter um dado(_senha_) antes de uma operação `POST` ou `PUT`.

{% highlight javascript %}
Resource.before('post', hash_password).before('put', hash_password)

function hash_password(req, res, next) {
  req.body.password = hash(req.body.password)
  next()
}
{% endhighlight %}

Você poderá acessar variáveis ​​locais em _templates rendered_ no aplicativo. Isso é útil para fornecer funções auxiliares para _templates_, bem como dados de nível de aplicativo. As variáveis ​​locais estão disponíveis no _middleware_ via `res.locals`:

```
res.locals.status_code => É o código de status retornado
res.locals.bundle => É o pacote de dados
```

Em todas as ligações antes e depois, você poderá modificar isso:

{% highlight javascript %}
Resource.after('get', (req, res, next) => {
  const tmp = res.locals.bundle.title // Permita trocar os campos do título e do ano
  res.locals.bundle.title = res.locals.bundle.year
  res.locals.bundle.year = tmp
  next() // Não se esqueça de ligar para a próxima!
})

Resource.after('recommend', do_something) // Executa após todos os verbos HTTP
{% endhighlight %}
_[Veja express docs][express]{:target="_blank"}_

Também podemos definir rotas detalhadas `/resources/route_name` como:

{% highlight javascript %}
Resource.route('moreinfo', {
    detail: true,
    handler: (req, res, next) => {
        // req.params.id holds the resource's id
        res.send("I'm at /resources/:id/moreinfo!")
    }
})
{% endhighlight %}

# Criando um exemplo simples

Antes de começar é necessário certificar-se que o **MongoDB**, **Nodejs** e o **NPM** estão instalados.

Criei um diretório com nome **node-resfull**.

{% highlight bash %}
$ mkdir node-restfull 
$ cd node-restfull
{% endhighlight %}

Utilizei o **npm init** para inicializar o projeto (_irá criar o package.json_)

{% highlight bash %}
$ npm init -y
{% endhighlight %}

Instalei os módulos que utilizei no projeto

{% highlight bash %}
$ npm i --save-dev body-parser express mongoose node-restful lodash
{% endhighlight %}

Criei um diretório com nome **api** e dentro o **category**

{% highlight bash %}
$ mkdir api
$ cd api
$ mkdir category
{% endhighlight %}

Também o diretório com nome **common** dentro do **api**

{% highlight bash %}
$ mkdir common
{% endhighlight %}

Crei o arquivo `errorHandler.js` dentro do **common** para tratamento de erros

{% highlight javascript %}
const _ = require('lodash')

module.exports = (req, res, next) => {
  const bundle = res.locals.bundle

  if(bundle.errors) {
    const errors = parseErrors(bundle.errors)
    res.status(500).json({errors})
  } else {
      next()
  }
}

const parseErrors = (nodeRestfulErrors) => {
  const errors = []
  _.forIn(nodeRestfulErrors, error => errors.push(error.message))
  return errors
}
{% endhighlight %}

Adicionei o arquivo `category.js` dentro do diretório **api/category**

{% highlight javascript %}
const restful = require('node-restful')
const mongoose = restful.mongoose

const categorySchema = new mongoose.Schema({
	description: { type: String },
	createAt: { type: Date, default: Date.now },
})

module.exports = restful.model('Category', categorySchema)
{% endhighlight %}

Adicionei o arquivo `categoryService.js`

{% highlight javascript %}
const Category = require('./category')
const errorHandler = require('../common/errorHandler')

Category.methods(['get', 'post', 'put', 'delete'])
Category.updateOptions({new: true, runValidators: true})
Category.after('post', errorHandler).after('put', errorHandler)

module.exports = Category
{% endhighlight %}

Criei o diretório **config** na raíz do projeto

{% highlight bash %}
$ mkdir config
{% endhighlight %}

Adicionei o arquivo `routes.js` e `database.js` dentro do **config**

`routes.js`
{% highlight javascript %}
const express = require('express')

module.exports = (server) => {

  const protectedApi = express.Router()
  server.use('/api', protectedApi)

  const Category = require('../api/category/categoryService')
  Category.register(protectedApi, '/categorys')
}
{% endhighlight %}

`database.js`
{% highlight javascript %}
const mongoose = require('mongoose')
mongoose.Promise = global.Promise

module.exports = mongoose.connect('mongodb://localhost/banco')
{% endhighlight %}

Na raíz do projeto criei o arquivo `server.js` e `loader.js`

`server.js`
{% highlight javascript %}
const port = 3004

const bodyParser = require('body-parser')
const express = require('express')
const server = express()

server.use(bodyParser.urlencoded({ extended: true}))
server.use(bodyParser.json())

require('./config/routes')(server)

server.listen(port, () => {
	console.log(`BACKEND is running on port ${port}.`)
})

module.exports = server
{% endhighlight %}

`loader.js`
{% highlight javascript %}
require('./server')
require('./config/database')
{% endhighlight %}

Prontinho! basta executar o comando abaixo

{% highlight bash %}
$ node loader.js
{% endhighlight %}

Para testar utilizei o `Postman`

{% highlight javascript %}
GET localhost:3004/api/categorys
GET localhost:3004/api/categorys/:id
POST localhost:3004/api/categorys
PUT localhost:3004/api/categorys/:id
DELETE localhost:3004/api/categorys/:id
{% endhighlight %}

# Conclusão

O fator que me levou a utilizá-la foi a flexibilidade de poder moldá-la como e quando desejei.

Espero que sirva de incentivo! :+1:

[Github node-restful](https://github.com/baugarten/node-restful){:target="_blank"}


[express]: http://expressjs.com/en/api.html