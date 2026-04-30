---
title: "Usando a gem httplog para uma melhor análise dos logs"
slug: "usando-a-gem-httplog-para-uma-melhor-analise-dos-logs"
published_at: "2023-04-19 12:01:16Z"
language: "pt-BR"
status: "published"
tags: ["braziliandevs", "ruby", "rails", "gem"]
---

Fala tropinha, tudo bom? Venho trazer nesse artiguinho minúsculo, um pouquinho sobre uma gem que tenho usado no dia-a-dia do trabalho para fazer alguns mapeamentos e documentação de aplicações legadas. A gem em questão é a **httplog**, e achei ela bem interessante por conseguir me exibir em detalhes os dados das requisições que são feitas para as mais diversas aplicações.

Para configurar ela, o primeiro passo é adicionar a gem no **Gemfile**:

```bash
gem 'httplog'
```

Depois disso basta rodar o seguinte comando no terminal, dentro da sua pasta do projeto rails:

```bash
bundle install
```

O último passo é você criar dentro da pasta **config/initializers** o arquivo **httplog.rb**:

```bash
HttpLog.configure do |config|

  # Enable or disable all logging
  config.enabled = true

  # You can assign a different logger or method to call on that logger
  config.logger = Logger.new($stdout)
  config.logger_method = :log

  # I really wouldn't change this...
  config.severity = Logger::Severity::DEBUG

  # Tweak which parts of the HTTP cycle to log...
  config.log_connect   = true
  config.log_request   = true
  config.log_headers   = false
  config.log_data      = true
  config.log_status    = true
  config.log_response  = true
  config.log_benchmark = true

  # ...or log all request as a single line by setting this to `true`
  config.compact_log = false

  # You can also log in JSON format
  config.json_log = false

  # Prettify the output - see below
  config.color = false

  # Limit logging based on URL patterns
  config.url_whitelist_pattern = nil
  config.url_blacklist_pattern = nil

  # Mask sensitive information in request and response JSON data.
  # Enable global JSON masking by setting the parameter to `/.*/`
  config.url_masked_body_pattern = nil

  # You can specify any custom JSON serializer that implements `load` and `dump` class methods
  # to parse JSON responses
  config.json_parser = JSON

  # When using graylog, you can supply a formatter here - see below for details
  config.graylog_formatter = nil

  # Mask the values of sensitive request parameters
  config.filter_parameters = %w[password]
  
  # Customize the prefix with a proc or lambda
  config.prefix = ->{ "[httplog] #{Time.now} " }
end
```

Prontinho, assim quando você subir a aplicação, ele carrega o **httplog** e você consegue ter uma visão um pouquinho melhor de cada request feita para outras aplicações. Tem como ainda naquele arquivinho que criamos nos **initializers** ajustar o texto exibido, cor, filtrar parâmetros, entre outros itens que estão disponíveis no arquivo de configuração.

![Imagem-do-Terminal-com-Logs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zxwwmd7m33pi05vgjtrp.png)

Caso você prefira não ficar vendo os logs pelo terminal, tem como ajustar para ele salvar em um arquivo de log, basta alterar a linha 7 do arquivo **httplog.rb** para:

```bash
config.logger = Logger.new('log/development.log')
```

Bom, por hoje é só, espero ter ajudado. Se você conhecer alguma outra gem que ajuda a ter uma melhor visão dos logs, comenta aí abaixo. Tmj!

**Link da gem:** [https://github.com/trusche/httplog](https://github.com/trusche/httplog)
