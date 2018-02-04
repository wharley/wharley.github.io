---
layout: post
title: Criando um CRUD com Angular e Nodejs + Sequelize
date: 2017-04-06 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post-3.jpg # Add image post (optional)
tags: [Blog, Angular]
author: Wharley Ornelas # Add name author (optional)
---
Para ficar completo o post anterior [(ORM Sequelize com database existente do Postgres no Nodejs)]({{site.baseurl}}/sequelize-com-database-existente-postgres/){:target="_blank"}, criei um crud para melhor visualização das informações.

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

{% highlight bash %}
$ ng g service setor.service
{% endhighlight %}

Importei no `@ngModule` do `app.module.ts`

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { SetorComponent } from './setor/setor.component';

//Adicionei esse import
import { SetorService } from './setor/setor.service';

@NgModule({
  declarations: [
    AppComponent,
    SetorComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  //Adicionei o service no providers - é um recurso que o Angular usa para fornecer (resultar, gerar) algo que queremos usar.
  //Se planeja usar um objeto várias vezes, por exemplo serviço Http em diferentes componentes, você pode pedir a mesma instância desse serviço (reutilizá-lo)
  providers: [SetorService],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Incluí o código no arquivo `setor.service.ts`

{% highlight typescript %}
import { Injectable } from '@angular/core';

//Importei o Http, Response, Headers, RequestOptions para requisições no Back-end
import { Http, Response, Headers, RequestOptions } from '@angular/http';
import { Observable } from 'rxjs/Observable';

import 'rxjs/add/operator/map';

@Injectable()
export class SetorService {

  headers: any = new Headers({ 'Content-Type': 'application/json' });
  options: any = new RequestOptions({ headers: this.headers });  

  constructor(private http: Http) { }

  //Função que irá fazer uma request no server para buscar a lista de setores
  getSetor() {
      return this.http.get('/listSetor')
                      .map(response => response.json())
  }

  //Função que irá fazer uma request no server para buscar apenas o setor de um determinado ID
  getSetorId(data) {

      return this.http.post('/byIdSetor', { id: data }, this.options)
                      .map(response => response.json())    
  } 

  //Função que irá fazer uma request no server para inserir um setor
  saveSetor(data) {

    return this.http.post('/saveSetor', data, this.options)
                    .map(response => response.json())
  }

  //Função que irá fazer uma request no server para dar update em um setor
  updateSetor(data) {

    return this.http.post('/updateSetor', data, this.options)
                    .map(response => response.json())
  }

  //Função que irá fazer uma request no server para deletar um setor
  delSetor(data) {

    return this.http.post('/deleteSetor', data, this.options)
                    .map(response => response.json())
  }     

}
{% endhighlight %}
"**Observable** _matriz cujos itens chegam de forma assíncrona ao longo do tempo. Observables ajudam a gerenciar dados assíncronos, como dados proveniêntes de um serviço back-end. Observables ​​são usados ​​dentro do próprio Angular, incluindo o sistema do evento do Angular e seu serviço do cliente do HTTP._"

Criei a interface adicionando o código no arquivo `setor.component.html`

{% highlight html %}
<section class="content">
    <div class="row">
        <div class="col-xs-12">
            <div class="box">
                <div class="box-header">
                    <div class="box-tools">
                        <div class="input-group">
                            <button class="btn btn-sm btn-primary" (click)="addSetor()"><i class="glyphicon glyphicon-plus"></i>Adicionar</button>
                            <input type="text" name="table_search" class="form-control input-sm pull-right" style="width: 150px;" placeholder="Pesquisa Setor" [(ngModel)]="filtro" />
                            <div class="input-group-btn">
                                <button class="btn btn-sm btn-default"><i class="fa fa-search"></i></button>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="box-body table-responsive no-padding">
                    <table class="table table-hover">
                        <tr>
                            <th>Código</th>
                            <th>Descrição</th>
                            <th>Ação</th>
                        </tr>
                        <tr *ngFor="let setor of obterSetores()">
                            <td>{{setor.id}}</td>
                            <td>{{setor.descricao}}</td>
                            <td>
                                <div class="buttons">
                                    <button class="btn btn-success btn-xs" (click)="editSetor(setor)">
                                                <span class="glyphicon glyphicon-edit" aria-hidden="true"></span>
                                                Editar
                                            </button>
                                    <button class="btn btn-danger btn-xs" (click)="delSetor(setor)">
                                                <span class="glyphicon glyphicon-trash" aria-hidden="true"></span>
                                                Exluir
                                            </button>
                                </div>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </div>
</section>
{% endhighlight %}
 - [(ngModel) - two-way data binding][ngmodel]{:target="_blank"} cria uma instância **FormControl** de um modelo de domínio e vincula a um elemento de controle de formulário.
 - [(click) - Event binding ( (event) )][click]{:target="_blank"} sintaxe de vinculação de eventos que consiste em um nome de evento de destino entre (parênteses) à esquerda de um sinal de igual e uma indicação de modelo citado à direita.

Criei um arquivo `setor.ts` que é uma classe com os atributos:

{% highlight typescript %}
export class Setor {
  id: number;
  descricao: string;
}
{% endhighlight %}

Adicionei as funções dos events no arquivo `setor.component.ts`

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

//Importei o Router devido os eventos click "editar", "excluir" e "adicionar"
import { Router } from '@angular/router';

//Importei o SetorService que foi criado anteriormente e a class Setor
import { SetorService } from './setor.service';
import { Setor } from './setor';

@Component({
  selector: 'app-setor',
  templateUrl: './setor.component.html',
  styleUrls: ['./setor.component.css']
})
export class SetorComponent implements OnInit {
  
  public setores: Setor[];
  filtro: string;
  private errorMessage: string;

  //Adicionei as instâncias SetorService, e Router
  constructor(private service: SetorService, private router: Router) { }

  ngOnInit() {
    this.onLoad()
  }

  onLoad() {
      //Estou invocando o método getSetor do SetorService
      this.service.getSetor()
                  .subscribe(data => this.setores = data,
                  error => this.errorMessage = <any>error);    
  }

  addSetor() {
    this.router.navigate(['/app-manutencao']) //Criarei o component "app-manutenção"
  }

  editSetor(setor) {
      this.router.navigate(['/app-manutencao', {id: setor.id}]) //Passando parâmetro pelo Router Navigate
  }

  delSetor(setor) {
     if (confirm(`Deseja excluir o setor ${setor.id} ?`)) {
        this.service.delSetor(setor)
                .subscribe(data => {
                    if(data){
                        alert('Registro excluído com sucesso')
                        this.onLoad()
                    }else{
                        alert('Registro não foi excluído')
                    }                    
                }, error => this.errorMessage = <any>error)
    }
  }  

  //Função para filtrar dados a partir de um input do usuário
  obterSetores() {

    if(this.setores === undefined || this.filtro === undefined || this.filtro.trim() === '' ) {
      return this.setores
    }

    return this.setores.filter((setor) => {

      if (setor.descricao.toLowerCase().indexOf(this.filtro.toLowerCase()) >= 0 ) {
        return true
      } else {
        return false
      }
    })

  }
}
{% endhighlight %}
 - [Router][route]{:target="_blank"} _permite a navegação de uma visão para a próxima, à medida que os usuários executam tarefas de aplicação_.
 - [Constructor][constructor]{:target="_blank"} _é inicializado pelo mecanismo de JavaScript, e o TypeScript nos permite dizer ao Angular quais dependências precisam ser mapeadas em relação a uma propriedade específica_.
 - [ngOnInit][ngoninit]{:target="_blank"} _ciclo de vida que é chamado após as propriedades de uma diretiva vinculadas a dados são inicializados_.

Criei a manutenção dos dados (Insert/Update), dentro do diretório _myApp/src/app/setor/_

{% highlight bash %}
$ ng g c manutencao
{% endhighlight %}

Dentro do diretório `myApp/src/app/setor/manutencao/` geraram-se os arquivos `manutencao.component.css`, `manutencao.component.html`, `manutencao.component.spec.ts`, `manutencao.component.ts` e importei no `app.module.ts`

Criei a interface adicionando o código no arquivo `manutencao.component.html`

{% highlight html %}
<section class="content">
    <div class="row">
        <div class="col-xs-12">
            <div class="box">
                <form name="form" (ngSubmit)="save()" #setor="ngForm">
                    <div class="box-header">
                        <div class="box-tools">
                            <div class="input-group">
                                <button type="submit" class="btn btn-sm btn-success" [disabled]="!setor.form.valid"><i name="save" id="save" class="glyphicon glyphicon-ok"></i>Salvar</button>
                                <button class="btn btn-sm btn-danger" (click)="cancel()"><i class="glyphicon glyphicon-remove"></i>Cancelar</button>
                            </div>
                        </div>
                    </div>
                    <div class="box box-primary">
                        <div class="box-header">
                            <h3 class="box-title">Informe os dados</h3>
                        </div>
                        <div class="form-group">
                            <input type="text" id="nameSetor" name="nameSetor" class="form-control" placeholder="Informe a descrição do setor" maxlength="100" [(ngModel)]="model.descricao" #nameSetor="ngModel" required/>
                            <span [hidden]="nameSetor.valid || nameSetor.pristine" class="alert alert-danger">descrição do setor</span>
                        </div>

                    </div>
                    <!-- /.box -->

                </form>
            </div>
        </div>
    </div>
</section>
{% endhighlight %}

Incluí as funções dos events do código anterior no arquivo `manutencao.component.ts`

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

import { Router, ActivatedRoute } from '@angular/router';
import { Subscription, Observable } from 'rxjs/Rx';

import { SetorService } from '../setor.service';
import { Setor } from '../setor';

@Component({
  selector: 'app-manutencao',
  templateUrl: './manutencao.component.html',
  styleUrls: ['./manutencao.component.css']
})
export class ManutencaoComponent implements OnInit {

  public model: Setor;
  private isNew: boolean = true;
  private subscription: Subscription;
  private errorMessage: string;  

  constructor(private service: SetorService, private router: Router, private route: ActivatedRoute) { }

  ngOnInit(): void {
    this.model = new Setor();

    this.subscription = this.route.params.subscribe((params: any) => {

        if (params.hasOwnProperty('id')) {
            this.isNew = false;
            this.service.getSetorId(params['id'])
                  .subscribe(data => this.model = data[0],
                  error => this.errorMessage = <any>error);
        } else {
            this.isNew = true;            
        }

      } 
    )
  }

  save() {

    let alerta = 'Registro gravado com sucesso!';
    let alError = 'Verificar os dados, não foi gravado';

    if(this.isNew){
        this.service.saveSetor(this.model)
                .subscribe(data => {
                    if(data){
                        alert(alerta)
                        this.router.navigate(['/app-setor'])
                    }else{
                        alert(alError)
                    }
                }, error => this.errorMessage = <any>error)
    }else{
        this.service.updateSetor(this.model)
                .subscribe(data => {
                    if(data){
                        alert(alerta)
                        this.router.navigate(['/app-setor'])
                    }else{
                        alert(alError)
                    }
                }, error => this.errorMessage = <any>error)
    }
    
  }

  cancel() {
    this.router.navigate(['/app-setor'])
  }

}
{% endhighlight %}

# Adicionando o Router...

Importei o **Router** no arquivo `app.module.ts` para funcionar o **_navigate_** e o **_router-link_** que irei incluir no **Menu**

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

//Acrescentei esse código
import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { SetorComponent } from './setor/setor.component';
import { SetorService } from './setor/setor.service';
import { ManutencaoComponent } from './setor/manutencao/manutencao.component';

//Acrecentei esse código
const appRoutes: Routes = [
  { path: 'app-setor', component: SetorComponent },
  { path: 'app-manutencao', component: ManutencaoComponent }
  { path: '**', redirectTo: '/' }
];

@NgModule({
  declarations: [
    AppComponent,
    SetorComponent,
    ManutencaoComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    RouterModule.forRoot(appRoutes, {useHash: true}) //Adicionei essa linha
  ],
  providers: [SetorService],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Modifiquei o `app.component.html`

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
                        //Alterado href="#" para [routerLink]
                        <li><a [routerLink]="['/app-setor']">Setor</a></li>
                        <li><a href="#">Produto</a></li>
                    </ul>
                </li>
            </ul>
        </div>
        <!-- /.navbar-collapse -->
    </div>
    <!-- /.container-fluid -->
</nav>

//Incluí o router-outlet
<router-outlet></router-outlet>
{% endhighlight %}
[router-outlet][routeroutlet]{:target="_blank"} _atua como um espaço reservado que o Angular preenche dinamicamente com base no estado atual do roteador_.

Para fazer o **produto** é o mesmo processo.

Criei o **Component produto** dentro do diretório **_myApp/src/app/_**

{% highlight bash %}
$ ng g c produto
{% endhighlight %}

_Gerou-se os arquivos `produto.component.css`, `produto.component.html`, `produto.component.spec.ts`, `produto.component.ts` e `importei no app.module.ts`_

Criei um arquivo `produto.service.ts` dentro do diretório **_src/app/produto/_**

{% highlight bash %}
$ ng g service produto.service
{% endhighlight %}

Importei no `@ngModule` do `app.module.ts` e adicionei no **Router** a rota do **produto**

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { SetorComponent } from './setor/setor.component';
import { SetorService } from './setor/setor.service';
import { ManutencaoComponent } from './setor/manutencao/manutencao.component';
import { ProdutoComponent } from './produto/produto.component';
import { ProdutoService } from './produto/produto.service';       //Incluí essa linha

const appRoutes: Routes = [
  { path: 'app-setor', component: SetorComponent },
  { path: 'app-manutencao', component: ManutencaoComponent },
  { path: 'app-produto', component: ProdutoComponent }            //Adicionei essa linha
  { path: '**', redirectTo: '/' }
];

@NgModule({
  declarations: [
    AppComponent,
    SetorComponent,
    ManutencaoComponent,
    ProdutoComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    RouterModule.forRoot(appRoutes, {useHash: true})
  ],
  providers: [SetorService, ProdutoService], //Adicionei ProdutoService
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Adicionei o código no arquivo `produto.service.ts`

{% highlight typescript %}
import { Injectable } from '@angular/core';

import { Http, Response, Headers, RequestOptions } from '@angular/http';
import { Observable } from 'rxjs/Observable';

import 'rxjs/add/operator/map';

@Injectable()
export class ProdutoService {

  headers: any = new Headers({ 'Content-Type': 'application/json' });
  options: any = new RequestOptions({ headers: this.headers });

  constructor(private http: Http) { }

  getProduto() {
      return this.http.get('/listProduto')
                      .map(response => response.json())
  }

  getProdutoId(data) {

      return this.http.post('/byIdProduto', { id: data }, this.options)
                      .map(response => response.json())    
  } 

  saveProduto(data) {

    return this.http.post('/saveProduto', data, this.options)
                    .map(response => response.json())
  }

  updateProduto(data) {

    return this.http.post('/updateProduto', data, this.options)
                    .map(response => response.json())
  }

  delProduto(data) {

    return this.http.post('/deleteProduto', data, this.options)
                    .map(response => response.json())
  } 

}
{% endhighlight %}

Criei a interface adicionando o código no arquivo `produto.component.html`

{% highlight html %}
<section class="content">
    <div class="row">
        <div class="col-xs-12">
            <div class="box">
                <div class="box-header">
                    <div class="box-tools">
                        <div class="input-group">
                            <button class="btn btn-sm btn-primary" (click)="addProduto()"><i class="glyphicon glyphicon-plus"></i>Adicionar</button>
                            <input type="text" name="table_search" class="form-control input-sm pull-right" style="width: 150px;" placeholder="Pesquisa Produto" [(ngModel)]="filtro" />
                            <div class="input-group-btn">
                                <button class="btn btn-sm btn-default"><i class="fa fa-search"></i></button>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="box-body table-responsive no-padding">
                    <table class="table table-hover">
                        <tr>
                            <th>Código</th>
                            <th>Descrição</th>
                            <th>Barra</th>
                            <th>Setor</th>
                            <th>Ação</th>
                        </tr>
                        <tr *ngFor="let produto of obterProdutos()">
                            <td>{{produto.id}}</td>
                            <td>{{produto.descricao}}</td>
                            <td>{{produto.barra}}</td>
                            <td>{{produto.Setor.descricao}}</td>
                            <td>
                                <div class="buttons">
                                    <button class="btn btn-success btn-xs" (click)="editProduto(produto)">
                                                <span class="glyphicon glyphicon-edit" aria-hidden="true"></span>
                                                Editar
                                            </button>
                                    <button class="btn btn-danger btn-xs" (click)="delProduto(produto)">
                                                <span class="glyphicon glyphicon-trash" aria-hidden="true"></span>
                                                Exluir
                                            </button>
                                </div>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </div>
</section>
{% endhighlight %}

Criei um arquivo `produto.ts` que é uma classe com os atributos:

{% highlight typescript %}
export class Produto {
  id: number;
  descricao: string;
  barra: string;
  id_setor: number;
}
{% endhighlight %}

Incluí as funções dos events no arquivo `produto.component.ts`

{% highlight bash %}
import { Component, OnInit } from '@angular/core';

import { Router } from '@angular/router';

import { ProdutoService } from './produto.service';
import { Produto } from './produto';

@Component({
  selector: 'app-produto',
  templateUrl: './produto.component.html',
  styleUrls: ['./produto.component.css']
})
export class ProdutoComponent implements OnInit {

  public produtos: Produto[];
  filtro: string;
  private errorMessage: string;  

  constructor(private service: ProdutoService, private router: Router) { }

  ngOnInit() {
    this.onLoad()
  }

  onLoad() {

      this.service.getProduto()
                  .subscribe(data => this.produtos = data,
                  error => this.errorMessage = <any>error);    
  }

  addProduto() {
    this.router.navigate(['/app-manutencao-produto'])
  }

  editProduto(produto) {
      this.router.navigate(['/app-manutencao-produto', {id: produto.id}])
  }

  delProduto(produto) {
        
     if (confirm(`Deseja excluir o produto ${produto.id} ?`)) {
        this.service.delProduto(produto)
                .subscribe(data => {
                    if(data){
                        alert('Registro excluído com sucesso')
                        this.onLoad()
                    }else{
                        alert('Registro não foi excluído')
                    }                    
                }, error => this.errorMessage = <any>error)
    }
  }  

  obterProdutos() {

    if(this.produtos === undefined || this.filtro === undefined || this.filtro.trim() === '' ) {
      return this.produtos
    }

    return this.produtos.filter((produto) => {

      if (produto.descricao.toLowerCase().indexOf(this.filtro.toLowerCase()) >= 0 ) {
        return true
      } else {
        return false
      }
    })

  }  

}
{% endhighlight %}

Criei a manutenção dos dados (Insert/Update), dentro do diretório _myApp/src/app/produto/_

{% highlight bash %}
$ ng g c manutencao
{% endhighlight %}

_Dentro do diretório **myApp/src/app/produto/manutencao/** geraram-se os arquivos `manutencao.component.css`, `manutencao.component.html`, `manutencao.component.spec.ts`, `manutencao.component.ts` e importei no `app.module.ts`_

Criei a interface adicionando o código no arquivo `manutencao.component.html`

{% highlight html %}
<section class="content">
    <div class="row">
        <div class="col-xs-12">
            <div class="box">
                <form (ngSubmit)="save()" #produtoForm="ngForm">
                    <div class="box-header">
                        <div class="box-tools">
                            <div class="input-group">
                                <button type="submit" class="btn btn-sm btn-success" [disabled]="!produtoForm.form.valid"><i name="save" id="save" class="glyphicon glyphicon-ok"></i>Salvar</button>
                                <button class="btn btn-sm btn-danger" (click)="cancel()"><i class="glyphicon glyphicon-remove"></i>Cancelar</button>
                            </div>
                        </div>
                    </div>
                    <div class="box box-primary">
                        <div class="box-header">
                            <h3 class="box-title">Informe os dados</h3>
                        </div>
                        <div class="form-group">
                            <input type="text" id="descricao" name="descricao" class="form-control" placeholder="Informe a descrição do produto" maxlength="100" [(ngModel)]="model.descricao" #descricao="ngModel" required/>
                            <div [hidden]="descricao.valid || descricao.pristine" class="alert alert-danger">Descrição do produto</div>
                        </div>

                        <div class="form-group">
                            <input type="text" id="barra" name="barra" class="form-control" placeholder="Informe o código de barras" maxlength="14" [(ngModel)]="model.barra" #barra="ngModel" required/>
                            <div [hidden]="barra.valid || barra.pristine" class="alert alert-danger">Código de barras</div>
                        </div>

                        <div class="form-group">
                            <select class="form-control" id="setor" required [(ngModel)]="model.id_setor" name="setor" #setor="ngModel">
                                    <option *ngFor="let setor of setores" [value]="setor.id">{{setor.descricao}}</option>
                            </select>
                            <div [hidden]="setor.valid || setor.pristine" class="alert alert-danger">Setor</div>
                        </div>

                    </div>
                    <!-- /.box -->

                </form>
            </div>
        </div>
    </div>
</section>
{% endhighlight %}

Adicionei as funções dos events do código anterior no arquivo `manutencao.component.ts`

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

import { Router, ActivatedRoute } from '@angular/router';
import { Subscription, Observable } from 'rxjs/Rx';

import { ProdutoService } from '../produto.service';
import { SetorService } from '../../setor/setor.service';
import { Produto } from '../produto';
import { Setor } from '../../setor/setor'

@Component({
  selector: 'app-manutencao-produto',            //Alterei o nome de app-manutencao para app-manutencao-produto não da conflito
  templateUrl: './manutencao.component.html',
  styleUrls: ['./manutencao.component.css']
})
export class ManutencaoProdutoComponent implements OnInit {

  public model: Produto;
  public setores: Setor[];
  private isNew: boolean = true;
  private subscription: Subscription;
  private errorMessage: string;    

  constructor(
      private service: ProdutoService, 
      private setorService: SetorService, 
      private router: Router, 
      private route: ActivatedRoute
  ) { }

  ngOnInit(): void {
    this.model = new Produto();

    this.onGetSetores()    

    this.subscription = this.route.params.subscribe((params: any) => {

        if (params.hasOwnProperty('id')) {
            this.isNew = false;
            this.service.getProdutoId(params['id'])
                  .subscribe(data => this.model = data[0],
                  error => this.errorMessage = <any>error);
        } else {
            this.isNew = true;            
        }

      } 
    )    
  }

  onGetSetores() {
      
    return this.setorService.getSetor()
                  .subscribe(data => this.setores = data,
                  error => this.errorMessage = <any>error);    
  }

  save() {

    let alerta = 'Registro gravado com sucesso!';
    let alError = 'Verificar os dados, não foi gravado';

    if(this.isNew){
        this.service.saveProduto(this.model)
                .subscribe(data => {

                    if(data){
                        alert(alerta)
                        this.router.navigate(['/app-produto'])
                    }else{
                        alert(alError)
                    }
                }, error => this.errorMessage = <any>error)
    }else{
        this.service.updateProduto(this.model)
                .subscribe(data => {
                    if(data){
                        alert(alerta)
                        this.router.navigate(['/app-produto'])
                    }else{
                        alert(alError)
                    }
                }, error => this.errorMessage = <any>error)
    } 
  }

  cancel() {
    this.router.navigate(['/app-produto'])
  }

}
{% endhighlight %}

Incluí a rota no arquivo `app.module.ts`

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { SetorComponent } from './setor/setor.component';
import { SetorService } from './setor/setor.service';
import { ManutencaoComponent } from './setor/manutencao/manutencao.component';
import { ProdutoComponent } from './produto/produto.component';
import { ProdutoService } from './produto/produto.service';
import { ManutencaoProdutoComponent } from './produto/manutencao/manutencao.component';

const appRoutes: Routes = [
  { path: 'app-setor', component: SetorComponent },
  { path: 'app-manutencao', component: ManutencaoComponent },
  { path: 'app-produto', component: ProdutoComponent },
  { path: 'app-manutencao-produto', component: ManutencaoProdutoComponent },   //Adicionei essa linha
  { path: '**', redirectTo: '/' }
];

@NgModule({
  declarations: [
    AppComponent,
    SetorComponent,
    ManutencaoComponent,
    ProdutoComponent,
    ManutencaoProdutoComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    RouterModule.forRoot(appRoutes, {useHash: true})
  ],
  providers: [SetorService, ProdutoService],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Modifiquei o `app.component.html`

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
                        <li><a [routerLink]="['/app-setor']">Setor</a></li>
                        //Alterado href="#" para [routerLink]
                        <li><a [routerLink]="['/app-produto']">Produto</a></li>
                    </ul>
                </li>
            </ul>
        </div>
        <!-- /.navbar-collapse -->
    </div>
    <!-- /.container-fluid -->
</nav>

<router-outlet></router-outlet>
{% endhighlight %}

Gerei o **build** com o comando:

{% highlight bash %}
$ ng build
{% endhighlight %}
_Gerou-se o diretório **dist** dentro do /myApp/_

Modifiquei o arquivo `setor.js` do back-end **_/modulos/setor/controller/_**

{% highlight javascript %}
exports.update = (req, res) => {

    const dados = req.body;

    model.Setor
        .update(dados, {
            where: {
                id: dados.id                //alterei essa linha
            }
        })
        .then((data) => }
            res.send(true);
        }).catch((error) => {
            console.log(error);
            res.send(false);
        });
};
{% endhighlight %}
{% highlight javascript %}
exports.delete = (req, res) => {

    const dados = req.body;

    model.Setor
        .destroy({
            where: {
                id: dados.id                //alterei essa linha
            }
        })
        .then((rowDeleted) => {
            res.send(true);
        }, (err) => {
            console.log(err);
            res.send(false);
        });
};
{% endhighlight %}

Modifiquei o arquivo `produto.js` do back-end **_/modulos/produto/controller/_**

{% highlight javascript %}
exports.update = (req, res) => {

    const dados = req.body;

    model.Produto
        .update(dados, {
            where: {
                id: dados.id                //alterei essa linha
            }
        })
        .then((data) => }
            res.send(true);
        }).catch((error) => {
            console.log(error);
            res.send(false);
        });
};
{% endhighlight %}
{% highlight javascript %}
exports.delete = (req, res) => {

    const dados = req.body;

    model.Produto
        .destroy({
            where: {
                id: dados.id                //alterei essa linha
            }
        })
        .then((rowDeleted) => {
            res.send(true);
        }, (err) => {
            console.log(err);
            res.send(false);
        });
};
{% endhighlight %}

Modifiquei o arquivo `app.js` do back-end que se encontra na raíz do projeto.

{% highlight javascript %}
'use strict'

const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');

const setor = require('./modulos/setor/setor');
const produto = require('./modulos/produto/produto');

const app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

//alterei a linha abaixo adicionando o diretório "dist"
app.use(express.static(path.join(__dirname, './myApp/dist/')));

setor.init(app);
produto.init(app);

module.exports = app;
{% endhighlight %}

Para rodar a **api**...

{% highlight bash %}
$ node bin/www
Listening on port 3000
{% endhighlight %}

# Conclusão

Com o **@angular/cli** é possivel criar um [single-page application (SPA)][singlepage]{:target="_blank"} simples em pouco tempo. Traduzir de uma linguagem para outra é principalmente uma questão de mudar a maneira como você organiza seu código e acessa APIs Angulares, qualquer coisa que você pode fazer com **Angular** no **TypeScript** você também pode fazer em **JavaScript**. 

Estou disponibilizando o projeto no [GitHub][github]{:target="_blank"} , acesse lá.

[npm]: https://www.npmjs.com/get-npm?utm_source=house&utm_medium=homepage&utm_campaign=free%20orgs&utm_term=Install%20npm
[node]: https://nodejs.org/en/
[angular]: https://angular.io/
[angularcli]: https://github.com/angular/angular-cli#installatio
[typescript]: https://www.typescriptlang.org/docs/tutorial.html
[bootstrap]: http://getbootstrap.com/
[jquerys]: https://jquery.com/
[fonts]: http://fontawesome.io/
[navbars]: https://getbootstrap.com/docs/3.3/components/#navbar
[ngmodel]: https://angular.io/api/forms/NgModel
[click]: https://angular.io/guide/template-syntax#!#event-binding
[route]: https://angular.io/api/router/Router
[constructor]: https://angular.io/guide/dependency-injection
[ngoninit]: https://angular.io/api/core/OnInit
[routeroutlet]: https://angular.io/api/router/RouterOutlet
[singlepage]: https://es.wikipedia.org/wiki/Single-page_application
[github]: https://github.com/wharley/front-end-Angular