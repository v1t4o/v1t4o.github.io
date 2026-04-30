---
title: "Implementação do Flipper para controle de features"
slug: "implementacao-do-flipper-para-controle-de-features"
published_at: "2023-06-02 15:19:42Z"
language: "pt-BR"
status: "published"
tags: ["braziliandevs", "ruby", "rails", "flipper"]
---

Fala tropinha, hoje venho trazer uma gemzinha braba também, que pode ajudar no dia-a-dia. A gem em questão é a Flipper, e ela é uma gem que fornece features flags que pode auxiliar você à controlar o acesso à determinadas features ainda em desenvolvimento.

## Instalação e Configuração

Para instalação, basta adicionar no seu Gemfile a gem flipper e a flipper-ui

```bash
gem 'flipper'
gem 'flipper-ui'
```

Além dessas duas, ainda devemos adicionar uma outra gem adapter, para termos onde guardar nossas flags. Vou utilizar aqui a do Redis, que tenho visto ser mais comum, mas pode ser que na sua empresa eles acabem optando por salvar no BD. Na documentação oficial, tem os adaptadores que conectam no banco e cada um tem sua doczinha bonitinha.

```bash
gem 'flipper-redis'
```

Depois disso, basta rodar o bundle. Na pasta config/initializers crie o arquivo flipper.rb com o seguinte conteúdo para configurarmos o Redis como o local onde será guardado as features flags:

```bash
require 'flipper/adapters/redis'

Flipper.configure do |config|
  config.adapter { Flipper::Adapters::Redis.new(Redis.new) }
end
```

Para configurar o acesso à UI do Flipper, você precisa fazer o processo de montagem da rota. Entre no arquivo routes.rb, e adicione a seguinte linha:

```bash
mount Flipper::UI.app(Flipper) => '/flipper'
```

Essa rota, se for necessário (e quase sempre é kkkk), tem como você protegê-la, na documentação tem algumas dicas. Não vou avançar muito aqui, pois depende um pouco de como cada empresa em que a gente trampa lida com isso.

## Interface para criação das features flags

Após subir sua aplicação, tente bater na rota /flipper e irá aparecer essa telinha:


![Home - Flipper](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cc2e5dnyu7pisi1iyacu.png)

Para adicionar alguma feature, basta clicar em Add Feature, e aparecerá a seguinte telinha:

![Added Feature in Flipper](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tqs5dugdpl83bchiciux.png)

Escolha um nome para a sua feature flag, em breve mostrarei como implementar isso no código. Ao prosseguir, aparece uma tela com algumas opções que temos em cada feature flag criada:


![Feature Flag Options](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2yt6nspvzkmabw9lulab.png)

O Flipper é uma mão na roda pra algumas coisas do dia-a-dia. Por exemplo, temos ali a opção de actors. Digamos que há um determinado fluxo da sua aplicação que tem gerado alguns erros para alguns clientes. Podemos usar essa opção para indicar os IDs desses clientes, e aplicar apenas para eles a solução de contorno. Podemos fazer isso também habilitando por grupos, como mostrado na tela acima.

Um outro cenário interessante de aplicação. Digamos que foi criado algum novo serviço para solucionar uma consulta, mas antes de fazer a troca em produção, você pensa em ir testando aos poucos. Tem como você habilitar uma certa porcentagem de usuários ou por tempo, e isso faz com que você consiga ir escalando essa quantidade, até perceber que pode mudar o fluxo na aplicação sem trazer grandes impactos aos clientes.

Tá, mas até agora só falei do Flipper, como isso se aplica no código? Bom, é bem tranquilo. Criei uma aplicação rails e gerei um scaffold para Car. Vou mostrar como isso poderia ser utilizado no controller, na parte de edit, e na view com o botão de deletar.

## Uso dentro do código e testes

Bom, estamos aqui no cars_controller.rb, e veja que é bem tranquilo para utilizar o Flipper.

```ruby
def edit
	render :show unless Flipper.enabled?(:edit)
end
```

No método de edit, eu coloquei uma verificação para que caso a feature flag :edit não esteja habilitada, eu renderize o show  do car. Caso esteja habilitada, ela prosseguirá o fluxo normal e com isso conseguirei editar o item.

Na view não é muito diferente esse processo, no arquivo show.html.erb, fiz algo bem parecido, envolvendo o botão de destroy em um if que só vai exibir se a feature flag estiver habilitada:

```ruby
<% if Flipper.enabled?(:destroy)%>
	<%= button_to "Destroy this car", @car, method: :delete %>
<% end %>
```

Um cuidado importante que se deve ter ao utilizar features flags do Flipper, é que ao deletar, ela deixa de existir em todo o sistema. Com isso, lembre-se de tirar a verificação da feature flag dos trechos de códigos onde você acabou usando por algum momento as flags.

E como ficaria ao criarmos testes? Bom, o processo é bem semelhante. Eu consigo, dentro dos testes, desabilitar ou habilitar alguma flag usando os métodos enable ou disable, e com isso simular os fluxos, veja abaixo um exemplo de teste:

```ruby
require 'rails_helper'

RSpec.describe "Feature Flags" do
 context "destroy a car" do
  it 'when feature flag is disabled' do
    Flipper.disable(:destroy)
    Car.create!(make: "Ford", model: "Ká", color: "Red", year: "2023", price: "R$ 30.000,00")
    visit cars_path
    click_on "Show this car"
    expect(page).not_to have_button("Destroy this car")
  end

  it 'when feature flag is enabled' do
    Flipper.enable(:destroy)
    Car.create!(make: "Ford", model: "Ká", color: "Red", year: "2023", price: "R$ 30.000,00")
    visit cars_path
    click_on "Show this car"
    expect(page).to have_button("Destroy this car")
  end
 end
end
```

## Considerações Finais

Bom, meu intuito era apresentar essa ferramenta, que pode ser muito útil no nosso dia-a-dia. Um uso interessante que notamos aqui na minha empresa atual, é de que em soluções críticas, que podem gerar impactos ao cliente, talvez testar a solução colocando uma feature flag seja mais interessante no ambiente de produção, uma vez que evita o processo de rollback. 

Além disso, com alguns controles do Flipper, é possível ir escalando a quantidade de clientes que podem ter acesso ou não à determinada feature, isso evita um impacto negativo no lançamento de uma nova feature. Observamos uma oportunidade interessante de usá-la também para reduzir quantidade de rollbacks e até como solução de contorno rápida para alguns problemas com prioridade alta.

## Fontes

https://github.com/jnunemaker/flipper

[Flipper Documentation](https://www.flippercloud.io/docs/introduction)
