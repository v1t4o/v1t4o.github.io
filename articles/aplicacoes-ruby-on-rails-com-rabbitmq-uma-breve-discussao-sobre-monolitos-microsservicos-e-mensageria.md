---
title: "Aplicações Ruby On Rails com RabbitMQ: uma breve discussão sobre monolitos, microsserviços e mensageria"
slug: "aplicacoes-ruby-on-rails-com-rabbitmq-uma-breve-discussao-sobre-monolitos-microsservicos-e-mensageria"
published_at: "2023-07-22 00:01:59Z"
language: "pt-BR"
status: "published"
tags: ["braziliandevs", "ruby", "rails", "microservices"]
---

Fala tropinha, tudo certo? Bom, mês passado fiz uma palestrinha técnica na firma, e me propus a falar um cadinho sobre RabbitMQ e a arquitetura de microsserviços, envolvendo nesse caso aplicações Ruby On Rails. Meu intuito aqui não é fazer uma super indagação sobre o tema, nem me aprofundar muito na implementação, mas talvez tentar fornecer um feijão com arroz sobre o tema, sobretudo pra quem nunca viu nada sobre.

## **O que é RabbitMQ? O que é mensageria?**

Tomo como ponto de partida a tentativa de explicar o que é o RabbitMQ, bom, ele é um software escrito em Erlang e foi lançado pela Rabbit Technologies em 2007. Ele é um software de mensageria e implementa o protocolo AMQP (Advanced Message Queuing Protocol). Guarda essa sigla pois ela será importante quando formos implementar nas apps em rubyzin. Um outro conceito importante é que o RabbitMQ é um Message Broker, o que isso quer dizer? Que ele é um servidor que apenas recebe as mensagens e alguém consome dele, não há um processamento via RabbitMQ. Isso é importante frizar pois no início podemos acabar confundindo o RabbitMQ com um Resque/Sidekiq da vida.

Sabendo disso, agora o interessante é pensarmos em que tipo de arquitetura se encaixa a aplicação do RabbitMQ, dito isto, vou entrar em uma discussão breve sobre o que seria a arquitetura monolítica e a de microsserviços.

## **Arquitetura Monolítica**

A arquitetura monolítica trata-se de um sistema que contempla vários módulos e funcionalidades. Se pensarmos em uma API monolítica, se trataria de uma API com várias rotas, que podem englobar pedidos, compras, produtos, clientes etc, tudo em um mesmo carinha. Isso acaba evidenciando que sistemas monolíticos envolvem processos altamente acoplados, executando como um único serviço. Além disso, em caso de pico de demanda, toda arquitetura que envolve a aplicação em si deverá ser escalada.

Seguindo alguns artigos que li, que deixo no final, apresento aqui os benefícios e desafios que a arquitetura monolítica apresenta

**Benefícios**

- Implantação através da cópia do pacote
- Compartilhamento de memória, espaço e recursos
- Desempenho em alguns casos, por conter em si todos os módulos

**Desafios**

- Alta complexidade ao longo do tempo
- Alto acoplamento
- Processo demorado de desenvolvimento e deploy
- Dificuldades na integração CI/CD
- Difícil escalonamento
- Uma falha derruba o sistema

## **Arquitetura de Microsserviços**

Nesse tipo de arquitetura, lembra aquela APIzona que a gente conversou mais a cima, então, imagina que vamos quebrar cada setorzinho em um serviço, ou seja, em vez de uma API que contempla todos aqueles módulos, teremos um carinha pra processar pedido, outro pra processar produto, outro para processar cliente etc, é como se cada microsserviço fosse um miniaplicativo que tem a própria arquitetura e lógica de negócios, com seu próprio banco de dados - o que garante um acoplamento flexível.

Novamente, a partir de alguns artigos que li, trago aqui um resumo dos benefícios e desafios da arquitetura de microsserviços.

**Benefícios**

- Mais rápido desenvolver, entender e manter
- Autonomia através do isolamento dos ciclos de vida
- Aplicativo poliglota
- Ciclos de lançamento independentes
- Escalonamento individual

**Desafios**

- Complexidade devido a ser distribuído
- Problema com transações distribuídas
- Teste abrangente torna-se mais complexo
- Implantação mais complexa
- Sobrecarga de operações
- Latência introduzida no tempo de resposta
- Banco de dados por serviço
- Instrumentar e monitorar ambiente
- Cada serviço tem sua regra de acesso
- Comunicação assíncrona

## **Exemplo de Uso**

Tendo isso em mente, vamos focar em um case de exemplo que eu bolei para a minha apresentação. Tentei idealizar uma aplicação legada em que utilizasse processos síncronos com jobs via Sidekiq/Resque, a ideia era mais ou menos essa aqui:


![Arquitetura de um monolito](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jhku7sy5mlrv43gjqj6u.png)

Veja que ali temos uma aplicação, que embora seja focada em distribuir as Notificações, ela acaba por ficar responsável por vários tipos de notificações: email, sms, push. Nesse caso, minha ideia, era pegar um desses tipos e quebrar em um serviço separado, suponhamos que a gama de emails disparados fosse muito alta. Na arquitetura em monolito, para escalar eu teria que aumentar o hardware de servidor ou de rede, ou até pensar na questão do banco de dados. Agora em microsserviço, veja como ficaria o exemplo:


![Arquitetura com microsserviço](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/epe34vxy73in7ffs1lhm.png)

Nesse caso, veja que eu tenho um Service Notifications responsável apenas pelo disparo dos e-mails e mesmo que eu tivesse um aumento na demanda por disparo de e-mails, não necessitaria escalar meu monolito por inteiro, agora eu escalo somente o meu microsserviço. Eu acho essa forma de pensar e partir pros microsserviços interessante. Geralmente, por hype, as pessoas tem vendido microsserviços como bala de prata para problemas de performance dentro das empresas, mas acredito que seja mais adequado pensar a arquitetura como uma solução se o contexto apresentar as condições necessárias. 

Essa forma mais cautelosa, fará que você apenas escale o que de fato necessitar de um legado, e ainda o mantiver ativo enquanto seu ciclo de vida permitir. Sair de um legado para uma arquitetura de microsserviços por inteiro, pode representar um desperdício de dinheiro em infraestrutura, uma vez que você pode ter segmentado vários módulos em microsserviços que não demandam tanto, representando máquinas ociosas, espaço em banco de dados alocado sem uso, entre outros impactos desnecessários. Enfim, o papo tá bom, mas como funciona o processo? Vamos lá, dividi em dois momentos a implementação em aplicações Ruby On Rails. Primeiro ponto é o Publisher e o segundo o Consumer.

## **Implementação**

### **Publisher**

Primeira coisa que fiz nesse exemplo acima, foi na aplicação Notifications incluir a gem do Bunny no Gemfile.

```jsx
gem 'bunny'
```

Após isso, rode o comando:

```jsx
bundle install
```

O segundo passo, é criarmos na pasta lib o arquivo bunny_client.rb com o seguinte conteúdo:

```jsx
# frozen_string_literal: true

class BunnyClient
	class << self

		# Cria a conexão com o RabbitMQ
		def connect!
			# Utiliza a variável CLOUDAMQP_URL para conectar no host indicado
			@connection = Bunny.new(ENV['CLOUDAMQP_URL'])
			@connection.start
			# Cria o canal
			@channel = @connection.create_channel
			# Cria nossa fila para onde vamos publicar as mensagens
			@fan_out = @channel.fanout('notifications_email.out')
			@connected = true
		end

		# Publica as mensagens na fila, informando a origem da app
		def push(payload)
			connect! unless @connected
			@fan_out.publish(payload, { app_id: 'notifications_email' })

			true
		end
	end
end
```

Crie um arquivo em config/initializers chamado bunny.rb com o seguinte conteúdo:

```jsx
require 'bunny'
require 'bunny_client'
```

No seu arquivo .env, não esqueça de adicionar a seguinte variável de ambiente:

```jsx
CLOUDAMQP_URL: amqp://localhost:5672
```

Dessa forma, dentro do fluxo da sua app, vamos supor que você tenha um model que após salvar localmente numa base de dados, você queira encaminhar para uma fila onde o microsserviço irá processar, você pode fazer algo como:

```jsx
class Notification < ApplicationRecord
	after_save :publish!
	
	def publish!
		message = {
			email: email,
			subject: subject,
			content: content    
		}
	
		# Aqui um exemplo, estamos utilizando o método do client que criamos acima
		BunnyClient.push(message.to_json)
	end
end
```

Para fazer funcionar na sua app, basta rodar o bom e velho **rails s** e bater na rota de criação por exemplo, para ver o funcionamento da publicação na fila.

### **Consumer**

No microsserviço, seguimos o mesmos passos iniciais que é incluir a gem do Bunny no Gemfile.

```jsx
gem 'bunny'
```

Após isso, rode o comando:

```jsx
bundle install
```

No seu arquivo .env, não esqueça de adicionar a seguinte variável de ambiente:

```jsx
CLOUDAMQP_URL: amqp://localhost:5672
```

Após isso, precisamos criar um arquivo na pasta bin chamado bunny_consumer.rb com o seguinte conteúdo:

```jsx
#!/usr/bin/env ruby

require File.expand_path('../config/environment', __dir__)

# Cria a conexão com o RabbitMQ
connection = Bunny.new(ENV['CLOUDAMQP_URL'])
connection.start
channel = connection.create_channel

# Cria a fila para processamento
queue = channel.queue('notifications_email.in', durable: true, auto_delete: false)

puts "\nStarting consumer!"

# Captura da fila out (que usamos na outra app) as mensagens existentes
fanout_name = 'notifications_email.out'
queue.bind(channel.exchange(fanout_name, type: 'fanout'))
puts "[consumer] #{queue.name} binds to #{fanout_name}"

# Inicia o processamento da mensagem
# Nesse nosso exemplo aqui, vamos utilizar o Modelo de Email, para o qual
# Vamos salvar local para ter um histórico do envio
# Mas poderia se tratar também de usar diretamente o mailer
queue.subscribe do |d_info, properties, payload|
	Email.create_and_send(JSON.parse(payload))
	puts "[consumer] #{queue.name} received #{properties[:type]}, from #{properties[:app_id]}: #{payload}\n"
end

begin
	while true
		sleep(3)
	end
rescue Interrupt
	connection.close
	puts "\nShutting down gracefully."
	exit
end
```

Rode o seguinte comando no terminal, para tornar esse arquivo executável:

```jsx
sudo chmod 755 bin/bunny_consumer.rb
```

Lá no nosso model de Email, a implementação da chamada a partir do BunnyConsumer ficaria dessa forma:

```jsx
class Email < ApplicationRecord
  def self.create_and_send(payload)
    message = new(
      email: payload['email'],
      subject: payload['subject'],
      content: payload['content']
    )
    message.save!
    send_message(message) 
  end

  def self.send_message(message)
    MessageMailer.with(message: message).send_email.deliver_now
  end
end
```

Veja que aqui eu consigo criar localmente o e-mail e logo após faço o envio do e-mail. Observe que esse microsserviço ele fica responsável apenas por processar isto, não há requisição ou qualquer outra coisa.

Por fim, para fazer funcionar esse carinha, basta rodar o comandinho:

```jsx
bin/./bunny_consumer.rb
```

Claro, você pode criar rotas para consultar esses dados no microsserviço se quiser, em vez da aplicação legada, basta criar um *Procfile* com ambos os comandos abaixo, e utilizar a gem do **Foreman**:

```jsx
web: bin/rails s -p 5002
service: bin/./bunny_consumer.rb
```

## **Conclusões**

Bom, espero que esse artiguinho ajude a introduzir a vocês um pouquinho dos conceitos de monolitos, microsserviços e mensageria com RabbitMQ. Volto a bater na tecla de que microsserviços não é bala de prata, é preciso considerar os contextos e necessidades. Entretanto, pode ser uma arquitetura que seja um trunfo em contextos de serviços que tenham uma demanda crescente e que vira e mexe necessitam de algum tipo de escalabilidade.

## **Fontes:**

https://aws.amazon.com/pt/microservices/

https://cloud.google.com/learn/what-is-microservices-architecture?hl=pt-br

[https://medium.com/@marcelomg21/arquitetura-de-microsserviços-bc38d03fbf64](https://medium.com/@marcelomg21/arquitetura-de-microsservi%C3%A7os-bc38d03fbf64)

https://github.com/ruby-amqp/bunny

https://world.hey.com/dhh/how-to-recover-from-microservices-ce3803cc
