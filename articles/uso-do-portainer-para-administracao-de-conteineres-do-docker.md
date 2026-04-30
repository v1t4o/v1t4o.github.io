---
title: "Uso do Portainer para administração de contêineres do Docker"
slug: "uso-do-portainer-para-administracao-de-conteineres-do-docker"
published_at: "2023-03-17 14:07:51Z"
language: "pt-BR"
status: "published"
tags: ["docker", "portainer", "braziliandevs"]
---

Fala minha tropa, tranquilidade? Nesse primeiro artigo aqui na [dev.to](http://dev.to) eu quero falar um pouquinho sobre o uso do **Portainer** para gerenciamento de contêineres do **Docker**. Essa dica foi um parceiro do meu trampo que me passou, vou compartilhar com geral pra ajudar no dia-a-dia.

Usar o **Docker** só via terminal é massa, mas devido à correria do dia-a-dia às vezes vale a pena ter disponível algo pra gerenciar de forma mais fácil. Dessa forma, o **Portainer** é uma boa solução.

Se você ainda não tem o dockerzão instalado na sua máquina, tem esse tutorial brabo de [Docker na Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04).

Pois bem, para tê-lo na sua máquina, o primeiro passo é você rodar esse comando para criar um volume para os dados do **Portainer**:

```bash
docker volume create portainer_data
```

Depois disso, tu precisa rodar o seguinte comando para baixar a imagem do **Portainer** e mapear os volumes para gerenciar o **Docker**:

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Para saber o que cada coisa significa, segue abaixo uma listinha com a descrição das paradas:

- **-d** - essa opção faz com que o container funcione em background
- **-p** - essa opção faz com que seja mapeada a porta 9443 para acesso via navegador na porta default do **Portainer** (9443) [segundo documentação]. Vale lembrar que você pode alterar essa porta, poderíamos ter utilizado 9000:9000, por exemplo.
- **--name** - é o nome do container a ser criado
- **--restart=always** - isso faz com que o container reinicie automaticamente
- **-v /var/run/docker.sock:/var/run/docker.sock** - mapeia o volume do container para gerenciamento dos outros contêineres do ambiente **Docker**
- **-v portainer_data:/data** - mapeia o volume referente aos dados do **Portainer** que criamos antes
- **portainer/portainer-ce:latest** - indica qual imagem do **Portainer** será utilizada

Bom, depois dos comandos terem rodados, é só você acessar via navegador na URL: [https://localhost:9443](https://localhost:9443/)

![Tela inicial do Portainer de configuração de usuário admin](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gv9t6x4ncww68w37qjgw.png)

No primeiro acesso, ele pedirá para você cadastrar uma senha pro admin. Basta preencher bonitinho e concluir pra ter acesso ao painelzinho do **Portainer**.

![Tela inicial do Portainer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gxmbihjsd4bj5e09pv7a.png)

No **Portainer** há várias funcionalidades, você tem uma dashboard informando os ambientes disponíveis:

![Listagem dos ambientes do Docker no Portainer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g9rfxvh105bv1ral6f19.png)

Você também tem uma dashboard por ambiente dos contêineres, volumes, imagens, redes criadas:

![Dashboard de um dos ambientes do Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fw2kddliz5ot2ze486ax.png)

Na listagem dos contêineres, você tem a opção também de visualizar os status, parar ou iniciar um container específico. Há também como adicionar novos contêineres por aqui. 

![Listagem dos contêineres de um ambiente do Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6xcnuyuvkhkkvadvdvfg.png)

Clicando em um deles, você consegue visualizar mais informações, como operações dos contêineres, status, acesso à logs e terminal.

![Visualização de informações de um container em específico](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e2wbxmcmi8hfm7lfwqof.png)

Bom, basicamente é isso, o intuito aqui era mais apresentar a ferramenta, que tem salvado muito no dia-a-dia do meu trampo. 

**Fontes**

[https://docs.portainer.io/start/install-ce/server/docker/linux](https://docs.portainer.io/start/install-ce/server/docker/linux)
