# fiapx-web

Front-end do **FIAP X — Sistema de Processamento de Vídeos**: login, upload de vídeos, acompanhamento
de status e download do zip de frames.

A SPA definitiva será **Vite + React + TypeScript**. O que existe hoje é o instrumento do spike de
upload assinado, descrito abaixo — ele é a semente da tela de upload e será substituído pela aplicação
Vite, não mantido em paralelo.

Os serviços vivem em [`fiapx-services`](https://github.com/CleytonOngaratto/fiapx-services) e as
dependências de runtime em [`fiapx-infra`](https://github.com/CleytonOngaratto/fiapx-infra).

## `spike-presigned/` — por que uma página, e não `curl`

O upload é um `PUT` em URL assinada feito **pelo navegador, direto no object storage**. Três coisas só
acontecem num navegador de verdade, e nenhuma delas o `curl` exercita:

1. o **preflight `OPTIONS`** com o `Origin` real, que o navegador manda sozinho;
2. o **host da URL assinada como o navegador o resolve** — a API assina com o endpoint que *ela*
   conhece, e o navegador pode não alcançá-lo;
3. o **`XMLHttpRequest.upload.onprogress`**, que é a barra de progresso.

Por isso a página é servida **noutra origem** (`:8090`) e não pela API (`:8080`).

### Rodar

```bash
# 1. dependências de runtime
cd ../fiapx-infra/dev && docker compose up -d

# 2. a API, noutro terminal
cd ../../fiapx-services && java -jar video-api/target/quarkus-app/quarkus-run.jar

# 3. esta página
cd ../fiapx-web/spike-presigned && node serve.mjs
```

Abra <http://localhost:8090>. Se a API estiver noutra porta, use
`http://localhost:8090/?api=http://localhost:8081`.

A página mostra os quatro passos com status e detalhe de cada um: pedir a URL assinada (CORS da API),
o `PUT` direto no storage (CORS do bucket, com a barra de progresso), o **ETag lido pelo navegador** e
a confirmação pelo servidor. Quando algo falha, ela diz **qual** das duas superfícies de CORS recusou —
que é a informação que o console do navegador esconde atrás de um "Failed to fetch".
