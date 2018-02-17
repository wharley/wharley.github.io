---
layout: post
title: Gerando relatórios na Web
date: 2017-05-21 13:32:20 +0300
description: Vindo de um universo de aplicações desktop, onde existem uma demanda muito grande de relatórios, assim veio a necessidade também na WEB.. # Add post description (optional)
img: post-4.png # Add image post (optional)
tags: [Blog, Javascript]
author: Wharley Ornelas # Add name author (optional)
---
No universo de aplicações **desktop** existe uma demanda muito grande de relatórios, também existente na **WEB**.

Pesquisando sobre as libs/frameworks uma delas me chamou a atenção pela simplicidade e de facilidade no aprendizado, o [pdfmake][pdfmaker]{:target="_blank"}, que é uma lib javascript onde possibilita imprimir PDFs diretamente no navegador ou delegue-o no backend do ecosistema NodeJS, podendo ser utilizada a mesma definição de documento em ambos os casos.

Neste post irei abordar a utilização do **pdfmake** do lado do client(**Front-end**), e o framework **Angular 4** no front-end, mas você pode usar o que desejar.

# Verificação de instalações!

Antes de começar é necessário certificar que o [Nodejs 6.9.5][nodejs]{:target="_blank"} (ou Superior), o [@angular/cli][angularcli]{:target="_blank"} e o **NPM 3** (ou Superior) estão instalados.

{% highlight bash %}
$ node -v 
$ npm -v
$ ng -v
{% endhighlight %}

Criei um diretório com nome **pdf-client**.

{% highlight bash %}
$ mkdir pdf-client 
$ cd pdf-client
{% endhighlight %}

Utilizei o angular/cli para poder criar o projeto

{% highlight bash %}
$ ng new myPdf
$ cd myPdf
{% endhighlight %}

Pronto! Nosso projeto está criado, para testar basta roda o comando:

{% highlight bash %}
$ ng serve
{% endhighlight %}

# Configurando o pdfmake!

Usei [cdnjs][cdn]{:target="_blank"}, [CDN ou Rede de Fornecimento de Conteúdo][rede]{:target="_blank"}, onde são hospedados todas as bibliotecas populares.

Alterei o `index.html` dentro do diretório **_/myPDF/src/_**

{% highlight html %}
<!doctype html>  
 <html>  
  <head>  
    <meta charset="utf-8">  
    <title>MyPdf</title>  
    <base href="/">  
    <meta name="viewport" content="width=device-width, initial-scale=1">  
    <link rel="icon" type="image/x-icon" href="favicon.ico">  
    <!-- Adicionei as duas linhas abaixo --> 
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.28/pdfmake.js"></script>  
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.28/vfs_fonts.js"></script>  
  </head>  
  <body>  
    <app-root>Loading...</app-root>  
  </body>  
 </html>
{% endhighlight %}

# Utilizando o pdfmake!

Alterei o `app.component.html` que se encontra no diretório **/myPdf/src/app/_**

{% highlight html %}
<div>  
    <h1>{{title}}</h1>  
    <button class="btn-default" (click)="openPdf()">Print default</button>  
    <button class="btn-primary" (click)="openPdfColors()">Print colorful</button>  
    <button class="btn-red" (click)="openPdfImage()">Print image</button>  
 <div>
{% endhighlight %}

Alterei o `app.component.css` que se encontra no diretório **_/myPdf/src/app/_**

{% highlight css %}
.btn-default {  
   background-color: #008CBA;  
   border: none;  
   color: white;  
   padding: 15px 32px;  
   text-align: center;  
   text-decoration: none;  
   display: inline-block;  
   font-size: 16px;  
 }  
 .btn-primary {  
   background-color: #4CAF50;  
   border: none;  
   color: white;  
   padding: 15px 32px;  
   text-align: center;  
   text-decoration: none;  
   display: inline-block;  
   font-size: 16px;  
 }  
 .btn-red {  
   background-color: #f44336;  
   border: none;  
   color: white;  
   padding: 15px 32px;  
   text-align: center;  
   text-decoration: none;  
   display: inline-block;  
   font-size: 16px;  
 }
{% endhighlight %}

Criei um `service` onde ficarão as funções e dados

{% highlight bash %}
$ ng g s app.service
{% endhighlight %}

Adicionei o código no `app.service.ts` dentro do diretório **_/myPdf/src/app/_**

{% highlight typescript %}
import { Injectable } from '@angular/core';  
 declare let pdfMake: any;  
 @Injectable()  
 export class AppService {  
  public products = [   
       {'code': 1, 'description': 'Nestlé Cookie 100g'},  
       {'code': 2, 'description': 'Danico biscuit 100g'},  
       {'code': 3, 'description': 'Rice Uncle Joao 5kg'},  
       {'code': 3, 'description': 'Coffee milk 1kg'}  
  ];  
  constructor() { }  
  getProduct() {  
       return this.products;  
  }  
  reportDefault(docDefinition) {  
       pdfMake.createPdf(docDefinition).open()  
  }  
 }
{% endhighlight %}

Importei o app.service.ts para o `app.module.ts` dentro do diretório **_/myPdf/src/app/_**

{% highlight typescript %}
 import { BrowserModule } from '@angular/platform-browser';  
 import { NgModule } from '@angular/core';  
 import { FormsModule } from '@angular/forms';  
 import { HttpModule } from '@angular/http';  
 import { AppComponent } from './app.component';  
 import { AppService } from './app.service';  
 @NgModule({  
  declarations: [  
   AppComponent  
  ],  
  imports: [  
   BrowserModule,  
   FormsModule,  
   HttpModule  
  ],  
  providers: [AppService],  
  bootstrap: [AppComponent]  
 })  
 export class AppModule { }
{% endhighlight %}

Alterei o `app.component.ts` dentro do diretório **_/myPdf/src/app/_**

{% highlight typescript %}
 import { Component } from '@angular/core';  
 import { AppService } from './app.service'  
 @Component({  
  selector: 'app-root',  
  templateUrl: './app.component.html',  
  styleUrls: ['./app.component.css']  
 })  
 export class AppComponent {  
  title = 'Pdfmake';  
  constructor(private service: AppService) {}  
  openPdf() {  
   let body: any = [];   
   const header: any = [  
    {text: 'Code', fontSize: 10, bold: true, alignment: 'center'},  
    {text: 'Description', fontSize: 10, bold: true, alignment: 'left'}  
   ]  
   body.push(header);  
   let products = this.service.getProduct().map((data) => {  
    let arr: any = [  
     {text: data.code, fontSize: 9, alignment: 'center'},  
     {text: data.description, fontSize: 9}  
    ]  
    body.push(arr);  
    return arr  
   })  
   const docDefinition = {  
    content: [  
     {text: 'Report of products', style: 'subheader'},  
     {  
      style: 'tableExample',  
      table: {  
       widths: ['auto', 300],  
       body: body  
      }  
     }  
    ],  
    styles: {  
     header: {  
      fontSize: 18,  
      bold: true,  
      margin: [0, 0, 0, 10]  
     },  
     subheader: {  
      fontSize: 16,  
      bold: true,  
      margin: [0, 10, 0, 5]  
     },  
     tableExample: {  
      margin: [0, 5, 0, 25]  
     },  
     tableHeader: {  
      bold: true,  
      fontSize: 13,  
      color: 'black'  
     }  
    },  
    defaultStyle: {  
     // alignment: 'justify'  
    }  
   }    
   this.service.reportDefault(docDefinition);  
  }   
  openPdfColors() {  
   let body: any = [];  
   const header: any = [  
    {text: 'Code', fontSize: 10, bold: true, alignment: 'center', fillColor: '#eeffee'},  
    {text: 'Description', fontSize: 10, bold: true, alignment: 'left', fillColor: '#eeffee'}  
   ]  
   body.push(header);  
   let products = this.service.getProduct().map((data) => {  
    let arr: any = [  
     {text: data.code, fontSize: 9, alignment: 'center', fillColor: '#eeeeff'},  
     {text: data.description, fontSize: 9, fillColor: '#eeeeff'}  
    ]  
    body.push(arr);  
    return arr  
   })  
   const docDefinition = {  
    content: [  
     {text: 'Report of products', style: 'subheader'},  
     {  
      style: 'tableExample',  
      table: {  
       widths: ['auto', 300],  
       body: body  
      }  
     }  
    ],  
    styles: {  
     header: {  
      fontSize: 18,  
      bold: true,  
      margin: [0, 0, 0, 10]  
     },  
     subheader: {  
      fontSize: 16,  
      bold: true,  
      margin: [0, 10, 0, 5]  
     },  
     tableExample: {  
      margin: [0, 5, 0, 25]  
     },  
     tableHeader: {  
      bold: true,  
      fontSize: 13,  
      color: 'black'  
     }  
    },  
    defaultStyle: {  
     // alignment: 'justify'  
    }  
   }    
   this.service.reportDefault(docDefinition);  
  }  
  openPdfImage() {  
   let body: any = [];  
   const header: any = [  
    {text: 'Code', fontSize: 10, bold: true, alignment: 'center', fillColor: '#eeffee'},  
    {text: 'Description', fontSize: 10, bold: true, alignment: 'left', fillColor: '#eeffee'},  
    {text: 'image', fontSize: 10, bold: true, alignment: 'left', fillColor: '#eeffee'}  
   ]  
   body.push(header);  
   let products = this.service.getProduct().map((data) => {  
    let arr: any = [  
     {text: data.code, fontSize: 9, alignment: 'center', fillColor: '#eeeeff'},  
     {text: data.description, fontSize: 9, fillColor: '#eeeeff'},  
     {image: 'data:image/jpeg;base64, 'AQUI EXISTE A IMAGEM, QUE SE ENCONTRA NO GITHUB', width: 25, fillColor: '#eeeeff'}  
    ]  
    body.push(arr);  
    return arr  
   })  
   const docDefinition = {  
    content: [  
     {text: 'Report of products', style: 'subheader'},  
     {  
      style: 'tableExample',  
      table: {  
       widths: ['auto', 300, 'auto'],  
       body: body  
      }  
     }  
    ],  
    styles: {  
     header: {  
      fontSize: 18,  
      bold: true,  
      margin: [0, 0, 0, 10]  
     },  
     subheader: {  
      fontSize: 16,  
      bold: true,  
      margin: [0, 10, 0, 5]  
     },  
     tableExample: {  
      margin: [0, 5, 0, 25]  
     },  
     tableHeader: {  
      bold: true,  
      fontSize: 13,  
      color: 'black'  
     }  
    },  
    defaultStyle: {  
     // alignment: 'justify'  
    }  
   }    
   this.service.reportDefault(docDefinition);  
  }   
 }
{% endhighlight %}

{% highlight bash %}
$ ng serve
{% endhighlight %}

![Report]({{site.baseurl}}/assets/img/reports.jpg)

# Conclusão
Com o **pdfmake** é possivel criar vários layouts de relatórios, inclusive existe o [playground][palyground]{:target="_blank"} com vários exemplos prontos e também é possível modelar o seu código antes de aplicar no seu projeto.

Até o momento está me atendendo muito bem nos meus projetos.

Estou disponibilizando o projeto no [GitHub][github]{:target="_blank"}, acesse lá.

[pdfmaker]: http://pdfmake.org/#/
[nodejs]: https://nodejs.org/en/
[angularcli]: https://github.com/angular/angular-cli#installatio
[cdn]: https://cdnjs.com/libraries/
[rede]: https://pt.wikipedia.org/wiki/Rede_de_fornecimento_de_conte%C3%BAdo
[playground]: http://pdfmake.org/playground.html
[github]: https://github.com/wharley/pdf-client