---
layout: post
title:  Conhecendo a biblioteca React
date:   2017-07-13 13:32:20 +0300
description: Estou apaixonado(React) e cada dia que passa meu amor aumenta, esse relacionamento está ficando muito sério. Será que vai da casamento ? duvida não!. # Add post description (optional)
img: post-5.png # Add image post (optional)
tags: [Blog, Javascript]
author: Wharley Ornelas # Add name author (optional)
---
Estou apaixonado(React) e cada dia que passa meu amor aumenta, esse relacionamento está ficando muito sério. Será que vai da casamento ? duvida não!

É isso mesmo galera, a cada dia que passa fico com os quatro pneus arriados por essa biblioteca fantástica.

# Afinal quem é ela React ?
[React][react]{:target="_blank"} é uma biblioteca **JavaScript** de código aberto para criar interfaces de usuário, desenvolvida por **Jordan Walke** um engenheiro do **Facebook**. Hoje é mantida pelo Facebook, pela comunidade de desenvolvedores individuais e outras empresas. De acordo com o serviço de análise de **JavaScript Libsore**, está sendo usado nos sites da **Netflix**, **Imgur**, **Relatório do Bleacher**, **Feedly**, **Airbnb**, **SeatGeek**, **HelloSign**, **Walmart** e outros.

# Características!
 - **Fluxo de dados unidirecional**:
   É um conjunto de valores imutáveis que são passados para o renderizador de um componente como propriedades sem sua tag **HTML**. É um componente que não pode modificar diretamente as propriedades passadas para ele, mas podem ser passadas funções de retorno de chamada que modificam valores.
 - **Virtual DOM**:
   É o uso de um **"Virtual Document Object Model"**. **React** cria um cache de estrutura de dados na memória, calcula as diferencas resultantes e em seguida atualiza eficientemente o **DOM** exibido pelo navegador. Isso permite que o programador escreva o código como se a página inteira fosse renderizada em cada alteração, enquanto o **React** somente representa **subComponents** que realmente mudam.
 - **JSX**
   Os components são escritos em **JSX**, uma sintaxe de extensão de **JavaScript** que permite citar **HTML** para redenrizar os subComponents, esta é uma extensão para o **JavaScript** como o **E4X(ECMAScript para xml)**, a sintaxe **HTML** é processada em chamadas **JavaScript** da estrutura do **React**. Os desenvolvedores também podem escrever em **JavaScript puro**.
   `app.js`
{% highlight javascript %}
import React, { Component } from 'react'  
class App extends component {   
    render() {  
        return (  
            <div>  
                <p>Hello World</p>
            </div>
        )  
    }  
}  
export default App 
{% endhighlight %}

# Ouvi falar que o React precisa de uma baita configuração!

Depende. Isso mesmo, existe o `create-react-app` que já faz esse processo todo para você em segundos, inclusive já com uma `App` funcionando. Só você começar a coda acrescentando seus componentes. Eu prefiro configurar o meu `webpack.config.js` para entender o funcionamento.

> "Afinal é uma biblioteca e não um framework..."

# Porque usar o React ?
Por ser uma biblioteca **JavaScript** de mecanismo dinâmico e também uma forma de reaproveitar **Components**, além da rapidez na renderização. E todos já sabem do meu amor por **JavaScript**.

# Hora da diversão

Antes de começar é necessário certificar que o **Nodejs** e o **NPM** estão instalados.

{% highlight bash %}
$ node -v 
$ npm5 -v
{% endhighlight %}

Criei um diretório com nome **app-react**.

{% highlight bash %}
$ mkdir app-react 
$ cd app-react
{% endhighlight %}

Utilizei o **npm init** para inicializar o projeto (_irá criar o package.json_)

{% highlight bash %}
$ npm init -y
{% endhighlight %}

Criei dois diretórios com nome **public** e **src**.

{% highlight bash %}
$ mkdir public src
{% endhighlight %}

E um arquivo `index.html` dentro do diretório **_app-react/public/_**

{% highlight html %}
<!DOCTYPE html>  
 <html>  
      <head>  
           <meta charset="utf-8">  
           <meta name="viewport" content="width=device-width, initial-scale=1">  
           <title>Welcome to React</title>  
           <link rel="stylesheet" href="app.css">  
      </head>  
      <body>  
           <div id="app" class="container"></div>  
           <script src="app.js"></script>  
      </body>  
 </html>
{% endhighlight %}

# Instalando todas as dependências:

Instalei todas as dependências para configuração do `webpack.config.js` incluíndo o **react** e **react-dom**.

{% highlight bash %}
$ npm5 i --save-dev babel-core@6.22.1 babel-loader@6.10.2 babel-plugin-html-attrs@2.0.0 babel-plugin-transform-object-rest-spread@6.22.0 babel-preset-es2015@6.22.0 babel-preset-react@6.22.0 extract-text-webpack-plugin@1.0.1 file-loader@0.9.0 style-loader@0.13.0 webpack@1.14.0 webpack-dev-server@1.16.2 react@15.4.2 react-dom@15.4.2
{% endhighlight %}

> [webpack][webpack]{:target="_blank"} é usado para compilar módulos de **JavaScript**.

Configurando o `package.json`

{% highlight json %}
"scripts": {  
   "dev": "webpack-dev-server --progress --colors --inline --hot",  
   "production": "webpack --progress -p"  
}
{% endhighlight %}

Criei o arquivo `webpack.config.js` dentro do diretório raíz **_app-react/_**

{% highlight javascript %}
const webpack = require('webpack')  
const ExtractTextPlugin = require('extract-text-webpack-plugin')  
module.exports = {  
      entry: './src/index.js',  
      output: {  
           path: __dirname + '/public',  
           filename: './app.js'  
      },  
      devServer: {  
           port: 3000,  
           contentBase: './public'  
      },  
      resolve: {  
           extensions: ['', '.js', '.jsx'],  
           alias: {  
                     modules: __dirname + '/node_modules'  
           }  
      },       
      plugins: [  
           new ExtractTextPlugin('app.css')  
      ],  
      module: {  
           loaders: [{  
                test: /.js[x]?$/,  
                loader: 'babel-loader',  
                exclude: /node_modules/,  
                query: {  
                     presets: ['es2015', 'react'],  
                     plugins: ['transform-object-rest-spread']  
                }  
           }, {  
                test: /\.css$/,  
                loader: ExtractTextPlugin.extract('style-loader', 'css-loader')  
           }, {  
                test: /\.woff|.woff2|.ttf|.eot|.svg*.*$/,  
                loader: 'file'  
           }]  
      }  
 }
{% endhighlight %}

Criei o diretório **_app-react/src/main/_**

{% highlight bash %}
$ mkdir main 
$ cd main
{% endhighlight %}

Adicionei o arquivo `app.js`

{% highlight javascript %}
import React from 'react'  
export default props => (  
    <div>  
       <h1>My first Component</h1>
    </div>
)
{% endhighlight %}

# O que é `props` ?

Conceitualmente, os componentes são como funções de **JavaScript**. Eles aceitam entradas arbitrárias(Chamadas de **props**) e retornam os elementos **React** descrevendo o que deve aparecer na tela.

Os componentes permitem que você divida a `UI` em peças independentes e reutilizáveis, pense em cada peça isoladamente(quebra cabeça)

Criei um arquivo `index.js` dentro do diretório **_app-react/src/_**

{% highlight javascript %}
import React from 'react'  
import ReactDOM from 'react-dom'  
import App from './main/app'  

ReactDOM.render(<App />, document.getElementById('app'))
{% endhighlight %}

# Pronto!
Só rodar o comando abaixo:

{% highlight bash %}
$ npm5 run dev
{% endhighlight %}


[react]: https://reactjs.org/
[webpack]: https://webpack.js.org/