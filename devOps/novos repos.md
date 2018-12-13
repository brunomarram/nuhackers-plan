# Passo a passo para criação de novos repositórios

---

#Ambiente de Desenvolvimento

**1 - Criar o repositório no Bitbucket**

Para criar o repositório, entramos no bitbucket e criamos o repositório dentro do projeto **nucontdev**.

**2 - Dar permissão aos membros do repositório**

Após a criação do mesmo, devemos ir nas configurações do repositório e selecionar quais usuários vão ter acesso de leitura/escrita aquele repositório.

**3 - Clonar e startar o repositório**

O primeiro passo é definir qual é o tipo de criação, para client:


```
meteor create "nome da aplicação"
```

Já no caso da apliação server:

```
meteor create --bare "nome da aplicação"
```
A partir desse passo, a aplicação já pode ser usada pelos desenvolvedores para enviar seus códigos fodas.

---

#Ambiente de Servidor

**4 - Definir e criar servidor no Bluemix**

Acessar o [bluemix](https://idaas.iam.ibm.com/idaas/mtfim/sps/authsvc?PolicyId=urn:ibm:security:authentication:asf:basicldapuser) e criar o servidor de acordo com as especificações definidas previamente, tais como:

- Banda que ira compor o hostname (metallica)
- Domínio da aplicação (Nucont.app)
- Localização **SEMPRE** *Washington 7*

**Obs.:** O processo de criação do servidor leva um tempo até que a IBM aloque tudo corretamente. Os próximos passos podem ser executados enquanto aguardamos a criação do mesmo para pegar as credenciais e o IP que serão necessárias para configuração do mup.

**4 - Configurar DNS no Cloudflare**

Com o IP do servidor em mãos, vamos configurar o DNS no [cloudflare](https://www.cloudflare.com/pt-br/). Se for uma aplicação que possui acesso direto com o cliente, temos que passar pelo cloudflare, ou seja:

- criar um registro A com o nome da banda apontando para o IP (sem passar pelo cloudflare).
- criar um registro CNAME com o endereço que o cliente irá acessar, apontando para o registro A (passando pelo cloudflare).

Se a aplicação não possuir acesso direto com cliente, basta usar o passo:

- criar um registro A com o nome da banda apontando para o IP (sem passar pelo cloudflare).

**5 - Criar uma nova aplicação no Kadira**

Após isso, criamos uma nova aplicação no Kadira com o seguinte formato:

    "nome do app" + "ambiente"
    Ex.: Nucont Production

Após criada a aplicação, pegamos o *APP_ID* e o *APP_SECRET* do mesmo. a URL usada pelo kadira é sempre:

    https://kadira.nucont.com:22022

Após isso, ainda na aplicação local, devemos rodar o seguinte comando para habilitar o kadira:

    meteor add meteorhacks:kadira

**6 - Criar o arquivo de mup**

Após isso, devemos criar o arquivo de configuração do [Meteor up](http://meteor-up.com/docs.html) dentro do diretório *.deploy/.${ambiente}*.

*mup.js*
```
module.exports = {
    servers: {
        thebeatles: {
            host: "thebeatles.nucont.com",
            username: process.env.USERNAME,
            password: process.env.PASSWORD
        }
    },

    meteor: {
        name: "nucont-client",
        path: "../../",

        servers: {
            thebeatles: {
                env: {

                }
            }
        },

        buildOptions: {
            serverOnly: true,
            debug: true
        },

        env: {
            ROOT_URL: "https://app-staging.nucont.com",
            MONGO_URL: process.env.MONGO_URL,
            DDP_DEFAULT_CONNECTION_URL: "https://app.nucont.com"
            
            KADIRA_APP_ID: process.env.KADIRA_APP_ID,
            KADIRA_APP_SECRET: process.env.KADIRA_APP_SECRET,
            KADIRA_OPTIONS_ENDPOINT: "https://kadira.nucont.com:22022",
        },

        docker: {
            image: "abernix/meteord:node-8.4.0-base"
        },

        deployCheckWaitTime: 180
    }
};
```

Bem como o json de configuração com as variáveis de ambiente utilizadas.

*settings.json*
```
{
    "nao-exposta": "chave de servidor"
    "public": {
        "exposta": "chave de client",
        "environment": "production",
        "url": "https://app.nucont.com"
    }
}
```

**7 - Fazer um mup setup no servidor**

Após criar os arquivos com as credenciais corretas, devemos executar o primeiro comando no servidor para preparar o mesmo para receber a aplicação, no caso, entre na pasta que possui os arquivos criados no passo anterior, e digite o seguinte comando:

    mup setup

Dessa forma o servidor estará pronto para receber a aplicação.

**8 - Habilitar o pipelines no Bitbucket**

Após habilitar o pipelines em *Settings -> Pipelines*, basta criar o arquivo de *bitbucket-pipelines.yml* na raiz do projeto. Um exemplo de arquivo:

*bitbucket-pipelines.yml*
```
image: brunomarram/meteor-mup-ci

pipelines:
    branches:
        production:
            - step:
                  name: Nucont-app to Production
                  deployment: production
                  caches:
                      - node
                  script:
                      - export USERNAME=$PRODUCTION_USERNAME
                      - export PASSWORD=$PRODUCTION_PASSWORD
                      - export MONGO_URL=$PRODUCTION_MONGO_URL
                      - meteor npm install
                      - cd .deploy/.production
                      - mup deploy
```

**9 - Adicionar as variáveis de repositório no Bitbucket**

Após isso, vamos novamente ao [Bitbucket](https://bitbucket.org) para podermos configurar as variáveis do repositório, que ficam em *Settings -> Pipelines -> Repository Variables*, lá, os nomes devem seguir o seguinte padrão:

    AMBIENTE_NOME
    Ex.: PRODUCTION_USERNAME

As informações sensíveis (senhas, credenciais, etc.) devem ser criptografadas e salvas dessa maneira. As demais variáveis podem ser salvas de forma simples, sem criptografia.

**10 - Criar branches definidas no bitbucket-pipelines.yml**

O último passo é criar as branches que foram definidas para se executar o CD (Continuous Delivery). Assim que as mesmas forem criadas, o processo de deploy irá iniciar e a aplicação estará pronta para ser utilizada no servidor em minutos :)

# Authors

-   **Bruno Marra** \- DevOps Manager \- [brunonucont](https://github.com/brunonucont)

# Thanks for

-   **Team NuHackers**