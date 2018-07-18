---
layout: post
title:  Migrando sua aplicação web para desktop
date:   2018-07-17 13:32:20 +0300
description: Fala galerinha :heart:. Estou a um bom tempo sem escrever post. Agradeço a todos que puxaram minha orelha por causa da falta de artigo no blog :smile:. # Add post description (optional)
img: post-9.jpg # Add image post (optional)
tags: [Blog, Nodejs]
author: Wharley Ornelas # Add name author (optional)
---

> Fala galerinha :heart:. Estou a um bom tempo sem escrever post. Agradeço a todos que puxaram minha orelha por causa da falta de artigo no blog :smile:.

Não vou ficar justificando... Desta vez irei mostrar como migrar sua aplicação web para destkop.

Javascript vem em uma evolução constante e hoje ela se encontra em todo universo:
  - Web ( Front-End )
  - Servidores em NodeJS ( Back-End )
  - Mobile
  - Jogos
  - Sistemas Operacionais
  - Hardwares
  - Desktop

E para fazer essa migração iremos usar o framework [ElectronJS][electron]{:target="_blank"}. Que é um framework de código aberto que permite desenvolver aplicações para desktop GUI usando componentes **Front-end** e **Back-end** originalmente criados para aplicações web: **Nodejs** para back-end e **Chromium** para o Front-end.
> Criado por **Cheng Zhao**, agora desenvolvido pelo **GitHub**.

Ele nos possibilita fazer essa migração em pouco tempo. Dependendo da aplicação até mesmo uns **5 minutos no máximo**. 

# Vamos a prática:

Está é uma aplicação web que fiz algum tempo atrás quando comecei estudar o **ReactJS**.

![Aplicação Web]({{site.baseurl}}/assets/img/application.gif)
> Para facilitar segue o link do código da [aplicação web][application]{:target="_blank"}, irei colocar o código somente da migração usando o `ElectronJS`.

# Vamos para a parte da mágica

Instalei o `ElectronJS` usando o **yarn**

```
yarn add electron
```

Na raíz do projeto `myAppDesktop`, crei o arquivo `main.js`

{% highlight javascript %}
const electron = require('electron')
// Controlar a vida da aplicação
const app = electron.app
// Criar janela do navegador nativo.
const BrowserWindow = electron.BrowserWindow

const path = require('path')
const url = require('url')

let mainWindow

function createWindow () {
  // Criar a janela do navegador
  mainWindow = new BrowserWindow({ width: 800, height: 600 })

  // Carrega o index.html do aplicativo
  mainWindow.loadURL(url.format({
    pathname: path.join(__dirname, '/public/index.html'),
    protocol: 'file:',
    slashes: true
  }))

  // Evento emitido quando a janela está fechada
  mainWindow.on('closed', () => {
    mainWindow = null
  })
} 

// Este método será chamado quando o Electron terminar a 
// inicialização e estiver pronto para criar janelas do navegador
// Algumas APIs só podem ser usadas depois que esse evento ocorrer
app.on('ready', createWindow)

// Sair quando todas as janelas estiverem fechadas
app.on('window-all-closed', () => {
  // No macOS é comum que os aplicativos e sua barra de menu 
  // permaneçam ativos até que o usuário saia explicitamente com Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

// Emitido quando o aplicativo é ativado
app.on('activate', () => {
  // No macOS é comum recriar uma janela no aplicativo quando o ícone 
  // dock é clicado e não há outras janelas abertas  
  if (mainWindow === null) {
    createWindow()
  }
})
{% endhighlight %}

Alterei o arquivo `package.json` conforme abaixo:
{% highlight json %}
{
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  },
}
{% endhighlight %}

# Pronto

```
yarn start
```
![Aplicação Desktop]({{site.baseurl}}/assets/img/desktop.gif)

# Gerando a aplicação para produção:

Instalei o [electron-packager][packager]

```
yarn add electron-package -D
```

Depois rodei o comando

```
electron-packager <sourcedir> <appname> --platform=<platform> --arch=<arch> [optional flags...]
```

Hoje existem vários aplicativos construídos no **Electronjs**:

![Apps-Desktop]({{site.baseurl}}/assets/img/electron-apps.png)
![Apps-Desktop-one]({{site.baseurl}}/assets/img/electron-apps-1.png)

Espero que sirva de incentivo! :+1:


[electron]: https://electronjs.org/
[packager]: https://github.com/electron-userland/electron-packager
[application]: https://github.com/wharley/myAppDesktop