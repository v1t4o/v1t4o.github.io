---
title: "Usando a gem Foreman"
slug: "usando-a-gem-foreman"
published_at: "2023-07-23 15:02:45Z"
language: "pt-BR"
status: "published"
tags: ["braziliandevs", "ruby", "rails", "tutorial"]
---

Fala minha tropa, suavidade? No último artiguinho, no fim dele eu comentei sobre a gem **Foreman**, pra não ficar no ar, resolvi escrever esse artiguinho curto sobre como utilizar essa *gem* que pode ser um adianto em muitos projetos para desenvolvimento local.

## Sobre o Foreman

O **Foreman** é uma *gem* que facilita a nossa vida quando se trata de vários processos de uma aplicação que necessitam ser executados. Um exemplo, aqui na firma tem um legadão que ele tem uma série de processos para além do **rails s.** Quando tentei executar a aplicação pela primeira vez, era meio cabaço, aí abri um três terminais só pra fazer funcionar. Só que aí, ela integrava com outro legadão que tinha outros 2 processos, e aí lá vai eu abrir mais dois terminais.

Bom, se eu tivesse usado o **Foreman**, teria usado apenas um terminal pra primeira app, e outro terminal pra segunda app, uma vez que eu consigo colocar dentro do arquivo de config do **Foreman** os vários serviços que necessito que estejam em execução apenas uma vez, sem precisar abrir vários terminais.

## Usando a gem

Primeiro passo, é você adicionar no seu projeto a gem do foreman

```ruby
gem 'foreman'
```

Depois disso, rode o comandinho mais famoso dos rubistas:

```ruby
bundle install
```

Aí na raiz do seu projeto, você precisará criar um arquivinho chamado ***Procfile***, onde você pode botar os processos que necessite, por exemplo:

```ruby
web:    rails s -p 3001
worker: bundle exec sidekiq
service: bin/./bunny_consumer.rb
```

Veja que sem o **Foreman**, eu teria que abrir três sessões do terminal para conseguir subir por completo minha app e seus serviços; já com a *gem*, para executar esses três carinhas basta no terminal eu rodar o seguinte comando pra mágica acontecer:

```ruby
foreman start
```

A carinha dele quando está em execução:


![Imagem do terminal com o Foreman em execução](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m69hr0ll5sj4rirk4sqi.jpeg)

## Conclusões

O **Foreman** ajuda muito a gente no ambiente de desenvolvimento, e acredito que seja uma mão na roda para quando chega devs novos no time, sobretudo os de nível júnior. Além disso, acho massa a questão do ***Procfile*** por funcionar meio como uma documentação do que é necessário pra app funcionar. Já peguei legados que tinha que ser meio Mãe Diná pra descobrir o que precisava rodar para aplicação funcionar. O **Foreman** também tem algumas paradas maneiras de exportação em produção, que ele cria uns serviços no **/etc/init** que pode ser uma mão na roda para gerenciar os serviços das aplicações no servidor. Ele também tem uma breve explicação sobre a questão de concorrência que pode ocorrer. Além dele, tem outros carinhas também, eu cheguei a mexer um cadinho com o **Overmind**, que é em *Golang*, e achei bem massa também. Mas me conta aí, que outro gerenciador de processos você conhece?

## Fontes

https://github.com/ddollar/foreman

http://blog.daviddollar.org/2011/05/06/introducing-foreman.html
