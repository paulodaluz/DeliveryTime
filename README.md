# DeliveryTime

API de estudo em **Node.js, TypeScript e Express** para estimar frete e prazo de entrega a partir de um pedido. Usa JWT, inicialização com TypeORM/MySQL e um serviço HTTP intermediário para a cotação.

[![TypeScript](https://img.shields.io/badge/TypeScript-project-3178C6)](https://www.typescriptlang.org/docs/)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)

## 📋 Table of contents

- [✨ Features](#features)
- [🧪 Technologies](#technologies)
- [🛠️ Environment setup](#environment-setup)
- [🚀 Getting started](#getting-started)
- [🔌 API usage](#api-usage)
- [🗂️ Project structure](#project-structure)
- [✅ Testing and quality](#testing-and-quality)
- [🔎 Troubleshooting](#troubleshooting)
- [🤝 Contributing](#contributing)
- [📜 License](#license)

<a id="features"></a>

## ✨ Features

- Emissão de JWT a partir de um email informado na URL.
- Cotação de prazo/frete a partir dos produtos e CEP de destino.
- Agregação de dimensões e peso antes da consulta HTTP ao serviço externo.
- Validação dos campos do pedido e respostas JSON.

<a id="technologies"></a>

## 🧪 Technologies

| Tecnologia | Versão declarada | Finalidade |
| --- | --- | --- |
| [Node.js](https://nodejs.org/) | Não definida no projeto | Runtime |
| [TypeScript](https://www.typescriptlang.org/docs/) | `^2.8.1` | Compilação para `js/` |
| [Express](https://expressjs.com/) | `^4.16.3` | Rotas HTTP |
| [TypeORM](https://typeorm.io/) | `^0.2.0` | Conexão MySQL |
| [MySQL driver](https://github.com/mysqljs/mysql) | `^2.15.0` | Acesso ao banco |
| [express-validator](https://express-validator.github.io/docs/) | `^5.3.1` | Validação das requisições |

As versões são faixas do manifesto, não versões resolvidas. Não há lockfile nem versão de Node em `engines`. Node **14.16.0** foi escolhido para a tentativa de reprodução do projeto legado; essa escolha não garante compatibilidade de dependências transitivas atuais. Atualizações de runtime/dependências exigem validação separada.
Também declara [Axios](https://axios-http.com/docs/intro) `^0.19.0` para HTTP e [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) `^8.5.1` para JWT. Não há testes Jest, Dockerfile ou documentação Swagger servida no código atual, embora o README anterior os citasse.

<a id="environment-setup"></a>

## 🛠️ Environment setup

Use Git, [nvm](https://github.com/nvm-sh/nvm) e Node.js **14.16.0**. Os comandos abaixo usam npm, fornecido com o Node.js. Não há lockfile versionado; use `npm install`, e não `npm ci`. A resolução das faixas de versão pode variar entre instalações.

```bash
nvm install 14.16.0
nvm use 14.16.0
git clone https://github.com/paulodaluz/DeliveryTime.git
cd DeliveryTime
npm install
```

### MySQL

Antes da execução, disponibilize uma instância MySQL própria e uma base vazia. Ajuste `ormconfig.json` com os dados locais:

```json
{
  "type": "mysql",
  "host": "127.0.0.1",
  "port": 3306,
  "username": "usuario_exemplo",
  "password": "senha_ficticia",
  "database": "ecommerce",
  "synchronize": true,
  "entities": ["js/entity/*.js"],
  "subscribers": ["js/subscriber/*.js"],
  "migrations": ["js/migration/*.js"],
  "cli": {
    "entitiesDir": "js/entity",
    "migrationsDir": "js/migration",
    "subscribersDir": "js/subscriber"
  }
}
```

Os valores acima são exemplos; não publique credenciais. `synchronize: true` permite ao TypeORM alterar o esquema: use uma base de desenvolvimento. O projeto não lê esse arquivo a partir de variáveis `.env`. O servidor só abre a porta após `createConnection()` concluir. Não há `prepare` ou hooks de Git configurados.

### Shipping service and authentication

`src/config/ShippingLocation.ts` contém `url` e `DepartureLocation` (código de serviço e CEP de origem). O destino configurado é um intermediário HTTP em `correios-server.herokuapp.com`, não uma chamada direta ao serviço SOAP dos Correios. Verifique a disponibilidade e a configuração desse serviço antes de usar a cotação.

O segredo JWT está em `src/config/auth.ts`. Configure uma chave de desenvolvimento própria localmente; não há leitura de variável de ambiente para substituí-la. Não reutilize nem divulgue a chave versionada.

<a id="getting-started"></a>

## 🚀 Getting started

```bash
./node_modules/.bin/tsc
npm start
```

`npm start` executa `tsc -w` e `nodemon js/index.js` em paralelo. `npm run watch` e `npm run server` também existem para execução separada. A porta é fixa em `3002`, sem leitura de `PORT`.

Com MySQL acessível e compilação concluída:

```bash
curl -X POST http://localhost:3002/autentificacao/usuario@example.com
```

Esse endpoint emite um token; não valida senha ou identidade do email. Não há rota de saúde.

<a id="api-usage"></a>

## 🔌 API usage

| Método | Caminho | Autenticação |
| --- | --- | --- |
| POST | `/autentificacao/:email` | Não exige token |
| POST | `/cotacao` | JWT diretamente em `Authorization`, sem `Bearer` |

Após obter um token no ambiente de desenvolvimento:

```bash
curl -X POST http://localhost:3002/cotacao \
  -H 'Content-Type: application/json' \
  -H 'Authorization: TOKEN_DO_AMBIENTE_LOCAL' \
  -d '{"request":{"address":{"postalCode":"01001000"},"order":{"orderTotal":100,"items":[{"product":{"taxCode":1,"length":20,"width":15,"height":5,"weight":0.5}}]}}}'
```

Dimensões são recebidas em centímetros e peso em quilogramas. O handler envia os dados ao serviço externo e retorna seu campo `response`. A consulta depende de disponibilidade e contrato do intermediário, que não foram confirmados nesta tarefa. O middleware atual registra o header de autenticação nos logs; use apenas token fictício/de desenvolvimento ao testar.

<a id="project-structure"></a>

## 🗂️ Project structure

| Caminho | Responsabilidade |
| --- | --- |
| `src/index.ts` | Conexão MySQL e início do servidor |
| `src/routes/` | Emissão de token e cotação |
| `src/controller/` | JWT e integração HTTP de frete |
| `src/services/ProductValidation.ts` | Validação do pedido |
| `src/config/` | Segredo JWT, origem e URL do serviço |
| `src/middlewares/auth.ts` | Verificação do JWT |
| `ormconfig.json` | Conexão e caminhos TypeORM |
| `js/` | Resultado da compilação (gerado) |

<a id="testing-and-quality"></a>

## ✅ Testing and quality

Não há scripts `test`, `lint` ou `build`. `./node_modules/.bin/tsc` é a verificação de compilação disponível. A instalação sem lockfile pode resolver declarações de tipos incompatíveis com TypeScript 2.x. Não confunda iniciar o watcher com compilar sem erros.

### Validation recorded

Em 11/11/2026, a instalação com Node 14.16.0/npm 6.14.11 concluiu. A compilação retornou código 2 por incompatibilidades nos tipos resolvidos em `@types/express-serve-static-core`. O JavaScript emitido foi iniciado, mas a conexão MySQL em `127.0.0.1:3306` retornou `ECONNREFUSED`; o servidor HTTP não abriu. As operações da API não foram verificadas. Nenhum serviço MySQL foi instalado.

<a id="troubleshooting"></a>

## 🔎 Troubleshooting

- `TypeORM connection error`: confira a instância MySQL, a base `ecommerce`, as credenciais e a porta em `ormconfig.json`.
- Erros de sintaxe em tipos de `node_modules`: as faixas abertas podem resolver tipos modernos incompatíveis com TypeScript 2.x; ajuste de dependências fica para uma alteração posterior.
- Cotação sem resposta: confira o intermediário configurado em `ShippingLocation.ts`; disponibilidade da API externa não é garantida.
- `401`: envie o JWT diretamente no header e utilize a mesma chave de assinatura/validação.

<a id="contributing"></a>

## 🤝 Contributing

Crie uma branch de `master`, documente o resultado de instalação/compilação e teste apenas com base e credenciais de desenvolvimento. Abra um PR descrevendo alterações e limitações da integração externa.

<a id="license"></a>

## 📜 License

Distribuído sob a licença **MIT**, preservada em [LICENSE](LICENSE).

---

<p align="center">Made with 💜 by <a href="https://github.com/paulodaluz">Paulo da Luz</a></p>
