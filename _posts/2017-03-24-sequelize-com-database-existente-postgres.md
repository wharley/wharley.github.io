---
layout: post
title:  ORM Sequelize com database existente do Postgres no Node.js
date:   2017-03-24 13:32:20 +0300
description: Devido experiências com banco de dados relacionais e percebendo que o mesmo vive no ecossistema do Nodejs, Resolvi estudar um pouco sobre ORM's que dão suporte em específico ao SGDB Postgres. # Add post description (optional)
img: post-2.jpg # Add image post (optional)
tags: [Blog, Nodejs]
author: Wharley Ornelas # Add name author (optional)
---
Neste post irei abordar a utilização do **Sequelize** com um database existente do **Postgres**.

Devido a experiências vivenciadas com banco de dados relacionais e percebendo que o mesmo vive no ecossistema do **Nodejs** resolvi estudar um pouco sobre **ORM's**, que dão suporte ao **SGDB Postgres**.

Após várias pesquisas cheguei no [ORM Sequelize][orm-sequelize]{:target="_blank"}, um ORM completo para banco de dados SQL, que atende ao **MySql**, **SQLite**, **MariaDB** e **MSSQL**.

# Verificações de instalações!

Antes de começar é necessário certificar que o [Nodejs][node-js]{:target="_blank"} está instalado. No **Postgres** iremos trabalhar com duas tabelas conforme figura abaixo:

![Database]({{site.baseurl}}/assets/img/database.jpg)
Tabelas **Produto** e **Setor**.

# Colocando a mão na massa

Para iniciar o projeto crie o diretório **database_existente**:

{% highlight bash %}
$ mkdir database_existente
$ cd database_existente
{% endhighlight %}

Utilizei o **npm init** para inicializar o projeto (_irá criar o **package.json**_)

{% highlight bash %}
$ npm init -y
{% endhighlight %}

Utilizei o **"-y"** para ignorar perguntas sobre o projeto.

Instalei o [Sequelize][Sequelize-install]{:target="_blank"} e também as dependências para o **Postgres**

{% highlight bash %}
$ npm i sequelize pg pg-hstore --save
{% endhighlight %}

E também instalei o framework [Express][Expressjs]{:target="_blank"}, que é um framework para aplicativo da web do **Nodejs** mínimo e flexível que fornece um conjunto robusto de recursos para aplicativos web e móvel.

{% highlight bash %}
$ npm i express --save
{% endhighlight %}

# Setup conexão Postgres

Criei uma pasta com nome **config**

{% highlight bash %}
$ mkdir config
$ cd config
{% endhighlight %}

Adicionei um arquivo [JSON][json]{:target="_blank"} com o nome `config.json`, ficando assim:

{% highlight json %}
{
    "development": {
        "dialect": "postgres",
        "port": 5432,
        "host": "localhost",
        "schema": "public",
        "database": "database_existente",
        "username": "postgres",
        "password": "postgres",
        "logging": false
    }
}
{% endhighlight %}

> logging == false desativando o console.log do **Sequelize**

# Estrutura de pastas

Model-view-controller (MVC).
 - **M**odelo um lugar para definir estruturas de dados e métodos para interagir com seu armazenamento de dados.
 - **V**isão um lugar para gerenciar tudo o que o usuário final vê em sua tela.
 - **C**ontrolador um lugar para receber solicitações de usuários, trazer dados do modelo e repassá-los para a visualização.

 Para ficar organizado, incluí uma pasta chamada **modulos** dentro da raíz do projeto

{% highlight bash %}
$ mkdir modulos
$ cd modulos
{% endhighlight %}

Conforme figura abaixo:

![estruturaPasta]({{site.baseurl}}/assets/img/estruturaPasta.jpg)

# Schemas

Adicionei o arquivo chamado `setor.js` dentro do diretório _modulos/setor/model/_

{% highlight javascript %}
'use strict'

module.exports = (sequelize, DataTypes) => {

    const Setores = sequelize.define('Setor', {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true,
            field: 'id'
        },
        descricao: {
            type: DataTypes.STRING,
            field: 'descricao'
        }
    }, {
        freezeTableName: true,
        schema: 'public',
        tableName: 'setor',
        timestamps: false
    })

    return Setores
}
{% endhighlight %}

O arquivo `produto.js` dentro do diretório _modulos/produto/model/_

{% highlight javascript %}
'use strict'

module.exports = (sequelize, DataTypes) => {

    var Produtos = sequelize.define('Produto', {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true,
            field: 'id'
        },
        descricao: {
            type: DataTypes.STRING,
            field: 'descricao'
        },
        barra: {
            type: DataTypes.STRING,
            field: 'barra'
        },
        id_setor: {
            type: DataTypes.INTEGER,
            field: 'id_setor'
        }
    }, {
        freezeTableName: true,
        schema: 'public',
        tableName: 'produto',
        timestamps: false,
        classMethods: {
            associate: (model) => {
                Produtos.belongsTo(model.Setor, { foreignKey: 'id_setor' })
            }
        }
    })

    return Produtos
}
{% endhighlight %}
> - **freezeTableName == true** sequelize não tentará alterar o nome DAO para obter o nome da tabela. Caso contrário, o nome do modelo será pluralizado;
 - **timestamps == false** Não adicionará as colunas createdAt e updatedAt timestamps para o modelo;
 - **classMethods** Fornece funções que são adicionadas ao modelo (**Model**). Se você substituir métodos fornecidos pelo sequelize, você pode acessar o método original usando this.constructor.prototype, This.constructor.prototype.find.apply (this, arguments);

# Carregamento dos modelos e conexão do Sequelize com Postgres

Incluí o arquivo `modelLoader.js` dentro do diretório _**utils/**_

{% highlight javascript %}
'use strict'

const env = process.env.NODE_ENV || 'development'          
const config = require('../config/config.json')[env]
const path = require('path')
let Sequelize = require('sequelize')

//Criando a conexão com o banco de dados de acordo com a configuração do config.json
let sequelize = new Sequelize(config.database, config.username, config.password, config)
let db = {}

//Criando um array do caminho dos modelos
const models = [
    '../modulos/setor/model/setor',
    '../modulos/produto/model/produto',
]

let l = models.length

//Irá importar os modelos para o sequelize
for (let i = 0; i < l; i++) {
    let model = sequelize.import(path.join(models[i]))
    db[model.name] = model
}

//Irá percorrer e separar apenas o objeto que contém a propriedade associate, sem o "associate" o sequelize não monta uma relação!
Object.keys(db).forEach((modelName) => {
    if ('associate' in db[modelName])
        db[modelName].associate(db)
})

db.sequelize = sequelize
db.Sequelize = Sequelize

module.exports = db
{% endhighlight %}

# Regras e solicitações dos usuários

Criei o arquivo `setor.js` dentro do diretório _**modulos/setor/controller/**_

{% highlight javascript %}
'use strict'

const model = require('../../../utils/modelLoader')

exports.read = (req, res) => {

    model.Setor.findAll(}

    }).then((data) => {

        res.send(data)

    }).catch((error) => {
        console.log(error)
        res.send(error)
    });
};

exports.insert = (req, res) => {

    const dados = req.body

    model.Setor
        .build(
            dados
        )
        .save()
        .then((data) => {
            res.send(true)
        }).catch((error) => {
            console.log(error)
            res.send(false)
        })
}

exports.update = (req, res) => {

    const dados = req.body

    model.Setor
        .update(dados, {
            where: {
                id: req.query.id
            }
        })
        .then((data) => {
            res.send(true)
        }).catch((error) => {
            console.log(error)
            res.send(false)
        })
}

exports.delete = (req, res) => {

    const dados = req.body

    model.Setor
        .destroy({
            where: {
                id: dados.params.id
            }
        })
        .then((rowDeleted) => {
            res.send(true)
        }, (err) => {
            console.log(err)
            res.send(false)
        })
}
{% endhighlight %}

O arquivo `produto.js` dentro do diretório _**modulos/produto/controller/**_

{% highlight javascript %}
'use strict'

const model = require('../../../utils/modelLoader')

exports.read = (req, res) => {

    model.Produto.findAll(}
        include: [
            { model: model.Setor }
        ]
    }).then((data) => {

        res.send(data)

    }).catch((error) => {
        console.log(error)
        res.send(error)
    })
}

exports.insert = (req, res) => {

    const dados = req.body

    model.Produto
        .build(
            dados
        )
        .save()
        .then((data) => {
            res.send(true)
        }).catch((error) => {
            console.log(error)
            res.send(false)
        })
}

exports.update = (req, res) => {

    const dados = req.body

    model.Produto
        .update(dados, {
            where: {
                id: req.query.id
            }
        })
        .then((data) => {
            res.send(true)
        }).catch((error) => {
            console.log(error)
            res.send(false)
        });
};

exports.delete = (req, res) => {

    const dados = req.body

    model.Produto
        .destroy({
            where: {
                id: dados.params.id
            }
        })
        .then((rowDeleted) => {
            res.send(true)
        }, (err) => {
            console.log(err)
            res.send(false)
        })
}
{% endhighlight %}

# Caminhos de rota

Criei os **roteamentos** onde definimos às solicitações do cliente. Para obter uma introdução a roteamento, consulte [Roteamento básico][router]{:target="_blank"}.

Incluí o arquivo `setor.js` dentro do diretório _**modulos/setor/routes/**_

{% highlight javascript %}
'use strict'

const express = require('express')
const router = express.Router()

const controller = require('../controller/setor')

router.get('/listSetor', controller.read)
router.post('/saveSetor', controller.insert)
router.post('/updateSetor', controller.update)
router.post('/deleteSetor', controller.delete)

module.exports = router
{% endhighlight %}

O arquivo `produto.js` dentro do diretório _**modulos/produto/routes/**_

{% highlight javascript %}
'use strict'

const express = require('express')
const router = express.Router()

const controller = require('../controller/Produto')

router.get('/listProduto', controller.read)
router.post('/saveProduto', controller.insert)
router.post('/updateProduto', controller.update)
router.post('/deleteProduto', controller.delete)

module.exports = router
{% endhighlight %}

Adicionei o arquivo `setor.js` dentro do diretório _**modulos/setor/**_

{% highlight javascript %}
'use strict'

const modelLoader = require('../../utils/modelLoader')

const routerSetor = require('./routes/setor')

const models = [
    '../modulos/setor/model/setor'
]

exports.init = (app) => {

    app.use('/', routerSetor)

}
{% endhighlight %}

O arquivo `produto.js` dentro do diretório _**modulos/produto/**_

{% highlight javascript %}
'use strict'

const modelLoader = require('../../utils/modelLoader')

const routerProduto = require('./routes/produto')

const models = [
    '../modulos/produto/model/produto'
]

exports.init = (app) => {

    app.use('/', routerProduto)

}
{% endhighlight %}

# Iniciando um servidor e escuta a porta 3000 por conexões

Instalei o [body-parser][body-parse]{:target="_blank"} que analisa os corpos de solicitação de entrada em um **middleware** antes de seus manipuladores, disponíveis na propriedade `req.body`.

{% highlight bash %}
$ npm i body-parser --save
{% endhighlight %}

Incluí o arquivo `app.js` dentro da raíz do projeto

{% highlight javascript %}
'use strict'

const express = require('express')
const path = require('path')
const bodyParser = require('body-parser')

const setor = require('./modulos/setor/setor')
const produto = require('./modulos/produto/produto')

const app = express()

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: false }))

setor.init(app)
produto.init(app)

module.exports = app
{% endhighlight %}

Criei uma pasta `bin` na raíz do projeto.

{% highlight bash %}
$ mkdir bin
{% endhighlight %}

E o arquivo `www` dentro do diretório _**bin/**_

{% highlight javascript %}
'use strict'

const app = require('../app')
const http = require('http')
const loader = require('../utils/modelLoader')

// Obter porta do meio ambiente e armazenamento no Express.

const port = normalizePort('3000')
app.set('port', port)

// Criar servidor HTTP.

let server = http.createServer(app)

// Escutar na porta todas as interfaces de rede.

server.listen(port)
server.on('error', onError)
server.on('listening', onListening)

// Normalizar uma porta em um número, string, ou falso.
function normalizePort(val) {
    const port = parseInt(val, 10)

    if (isNaN(port)) {
        // pipe nomeado
        return val
    }

    if (port >= 0) {
        // número da porta
        return port
    }

    return false
}

// Ouvinte de eventos para o servidor HTTP evento "erro".
function onError(error) {
    if (error.syscall !== 'listen') {
        throw error
    }

    const bind = typeof port === 'string' ?
        'Pipe ' + port :
        'Port ' + port

    // Ouvir erros com mensagens amigáveis
    switch (error.code) {
        case 'EACCES':
            console.error(bind + ' exige privilégios elevados')
            process.exit(1)
            break
        case 'EADDRINUSE':
            console.error(bind + ' já está em uso')
            process.exit(1)
            break
        default:
            throw error
    }
}

// ouvir eventos para o servidor HTTP.
function onListening() {
    const addr = server.address()
    const bind = typeof addr === 'string' ?
        'pipe ' + addr :
        'port ' + addr.port
    console.log('Listening on ' + bind)

}
{% endhighlight %}

# Prontinho...

Para rodar a **api**...

{% highlight bash %}
database_existente$ node bin/www
Listening on port 3000
{% endhighlight %}

Estou disponibilizando o projeto no [GitHub][fontes-github]{:target="_blank"} e também a documentação das requisições para testes, acesse lá e baixe o mesmo.

Para testar as requisições, utilizo a ferramenta [Postman][postman]{:target="_blank"}.

[orm-sequelize]: https://sequelize.readthedocs.io/en/v3/
[node-js]:   https://nodejs.org/en/
[Sequelize-install]: http://docs.sequelizejs.com/manual/installation/getting-started.html
[Expressjs]: http://expressjs.com/pt-br/
[json]: http://www.json.org/json-pt.html
[router]: http://expressjs.com/pt-br/guide/routing.html
[body-parse]: https://github.com/expressjs/body-parser
[fontes-github]: https://github.com/wharley/database_existente
[postman]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop
