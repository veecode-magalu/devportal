# Magalu DevPortal

Instruções para execução local do DevPortal servindo o catálogo de templates para a Magalu Cloud.

## Pré-requisitos

- VKDR (VeeCode Kubernetes Development Runtime)
- Docker Desktop (ou similar)

## Passos

1. Inicie seu cluster VKDR local com um ingress controller *default* (vamos usar o Kong porque sua UI administrativa é muito útil para depuração):

```bash

vkdr infra start --http 80 --https 443
vkdr kong install --default-ic -d localdomain

```

2. Instalando outras ferramentas necessárias:

```bash

vkdr vault install -d localdomain -s
vkdr vault init

```

3. Registre a URL do DevPortal como OAuth App no Github

- Acesse https://github.com/settings/developers
- Crie uma nova OAuth App chamada "devportal-localdomain"
  - URL: https://devportal.localdomain/
  - Callback URL: https://devportal.localdomain/api/auth/github/handler/frame
  - Anote ClientID
  - Gere Client secret e o anote também

4. Crie um token de acesso para o GitHub

- Acesse https://github.com/settings/developers
- Crie um novo fine grained token chamado "magalu-token" e anote seu valor
- Conceda acesso R/W a todos os repos da organização "veecode-magalu" (ou àquela que você desejar)

5. Instale o DevPortal apontando para o catálogo de Magalu Cloud:

A maneira mais fácil de instalar o DevPortal é também usando o VKDR:

```bash

export GITHUB_TOKEN=xxxx
export GITHUB_CLIENT_ID=xxxx
export GITHUB_CLIENT_SECRET=xxxx
vkdr devportal install -d localdomain \
    --github-token $GITHUB_TOKEN \
    --github-client-id $GITHUB_CLIENT_ID \
    --github-client-secret $GITHUB_CLIENT_SECRET
```

Alternativamente você pode instalar o DevPortal usando o Helm diretamente:

```bash

export GITHUB_TOKEN=xxxx
export GITHUB_CLIENT_ID=xxxx
export GITHUB_CLIENT_SECRET=xxxx

#vkdr devportal install -d localdomain
helm upgrade platform-devportal --install --wait --timeout 10m veecode-platform/devportal \
    --create-namespace -n platform -f values.yaml \
    --set "integrations.github[0].token=${GITHUB_TOKEN}" \
    --set "auth.providers.github.clientId=${GITHUB_CLIENT_ID}" \
    --set "auth.providers.github.clientSecret=${GITHUB_CLIENT_SECRET}"

```
