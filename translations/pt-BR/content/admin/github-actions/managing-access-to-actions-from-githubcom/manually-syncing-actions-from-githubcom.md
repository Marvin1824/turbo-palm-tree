---
title: Sincronização manual de ações do GitHub.com
intro: 'Para usuários que precisam acessar as ações a partir de {% data variables.product.prodname_dotcom_the_website %}, você pode sincronizar ações específicas para sua empresa.'
redirect_from:
  - /enterprise/admin/github-actions/manually-syncing-actions-from-githubcom
  - /admin/github-actions/manually-syncing-actions-from-githubcom
versions:
  ghes: '*'
  ghae: next
type: tutorial
topics:
  - Actions
  - Enterprise
shortTitle: Sincronizar ações manualmente
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}
{% data reusables.actions.ae-beta %}

{% data reusables.actions.enterprise-no-internet-actions %}

{% ifversion ghes or ghae-next %}

A abordagem recomendada de habilitar o acesso a ações a partir de {% data variables.product.prodname_dotcom_the_website %} é permitir o acesso automático para todas as ações. Você pode fazer isso usando {% data variables.product.prodname_github_connect %} para integrar {% data variables.product.product_name %} com {% data variables.product.prodname_ghe_cloud %}. Para obter mais informações, consulte "[Habilitar o acesso automático às ações de {% data variables.product.prodname_dotcom_the_website %} usando o {% data variables.product.prodname_github_connect %}](/enterprise/admin/github-actions/enabling-automatic-access-to-githubcom-actions-using-github-connect)".

No entanto, se você quiser um controle mais rigoroso sobre quais as ações são permitidas na sua empresa, você{% else %}Você{% endif %} poderá seguir este guia para usar a ferramenta de código aberto de {% data variables.product.company_short %} [`ação-sincronização`](https://github.com/actions/actions-sync) para sincronizar repositórios de ações individuais de {% data variables.product.prodname_dotcom_the_website %} para a sua empresa.

## Sobre a ferramenta `actions-sync`

A ferramenta `actions-sync` deve ser executada em uma máquina que pode acessar a API de {% data variables.product.prodname_dotcom_the_website %} e sua API da instância do {% data variables.product.product_name %}. A máquina não precisa estar conectada a ambos ao mesmo tempo.

Se sua máquina tiver acesso aos dois sistemas ao mesmo tempo, você poderá fazer a sincronização com um único comando de `actions-sync`. Se você só puder acessar um sistema de cada vez, pode usar os comandos `actions-sync pull` e `push`.

A ferramenta `actions-sync` só pode fazer download de ações de {% data variables.product.prodname_dotcom_the_website %} armazenadas em repositórios públicos.

{% ifversion ghes > 3.2 or ghae-issue-4815 %}
{% note %}

**Observação:** A ferramenta `actions-sync` destina-se a ser usada em sistemas em que {% data variables.product.prodname_github_connect %} não está habilitado. Se você executar a ferramenta em um sistema com {% data variables.product.prodname_github_connect %} habilitado, você poderá ver o erro `O repositório <repo_name> foi desativado e não pode ser reutilizado`. Isso indica que um fluxo de trabalho usou essa ação diretamente em {% data variables.product.prodname_dotcom_the_website %} e o namespace está desativado em {% data variables.product.product_location %}. Para obter mais informações, consulte "[Desativação automática de namespaces para ações acessadas em {% data variables.product.prodname_dotcom_the_website%}](/admin/github-actions/managing-access-to-actions-from-githubcom/enabling-automatic-access-to-githubcom-actions-using-github-connect#automatic-retirement-of-namespaces-for-actions-accessed-on-githubcom)".

{% endnote %}
{% endif %}

## Pré-requisitos

* Antes de usar a ferramenta `actions-sync`, você deve garantir que todas as organizações de destino existem na sua empresa. O exemplo a seguir demonstra como sincronizar ações com uma organização com o nome de `synced-actions`. Para obter mais informações, consulte "[Criar uma nova organização do zero](/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch)".
* Você deve criar um token de acesso pessoal (PAT) na sua empresa que pode criar e gravar em repositórios nas organizações de destino. Para obter mais informações, consulte[Criando um token de acesso pessoal](/github/authenticating-to-github/creating-a-personal-access-token)."{% ifversion ghes %}
* Se você deseja sincronizar as ações empacotadas na organização das `ações` em {% data variables.product.product_location %}, você deverá ser proprietário da organização das `ações`.

  {% note %}

  **Observação:** Por padrão, até os administradores do site não são proprietários das ações agrupadas ``.

  {% endnote %}

  Os administradores dos sites podem usar o comando `ghe-org-admin-promote` no shell administrativo para promover um usuário para ser proprietários da organização das `ações` empacotadas. Para obter mais informações, consulte "[Acessar o shell administrativa (SSH)](/admin/configuration/accessing-the-administrative-shell-ssh)" e "[`ghe-org-admin-promote`](/admin/configuration/command-line-utilities#ghe-org-admin-promote)".

  ```shell
  ghe-org-admin-promote -u <em>USERNAME</em> -o actions
  ```{% endif %}

## Example: Using the `actions-sync` tool

Este exemplo demonstra o uso da ferramenta `actions-sync` para sincronizar uma ação individual a partir de {% data variables.product.prodname_dotcom_the_website %} para a instância de uma empresa.

{% note %}

**Observação:** Este exemplo usa o comando `actions-sync sync`, que exige acesso simultâneo à API de {% data variables.product.prodname_dotcom_the_website %} e à API da instância empresarial da sua máquina. Se você puder acessar apenas um sistema de cada vez, você poderá usar os comandos `actions-sync pull` e `push`. Para obter mais informações, consulte [README de `actions-sync`](https://github.com/actions/actions-sync#not-connected-instances).

{% endnote %}

1. Faça o download e extraia a versão mais recente [`actions-sync`](https://github.com/actions/actions-sync/releases) para o sistema operacional da sua máquina.
1. Crie um diretório para armazenar arquivos de cache para a ferramenta.
1. Run the `actions-sync sync` command:

   ```shell
   ./actions-sync sync \
     --cache-dir "cache" \
     --destination-token "aabbccddeeffgg" \
     --destination-url "https://my-ghes-instance" \
     --repo-name "actions/stale:synced-actions/actions-stale"
   ```

   O comando acima usa os seguintes argumentos:

   * `--cache-dir`: O diretório de cache na máquina que está executando o comando.
   * `--destination-token`: Um token de acesso pessoal para a instância empresarial de destino.
   * `--destination-url`: A URL da instância empresarial de destino.
   * `--repo-name`: O repositório da ação a ser sincronizado. Ele aceita o formato de `owner/repository:destination_owner/destination_repository`.

     * O exemplo acima sincroniza o repositório [`actions/stale`](https://github.com/actions/stale) com o repositório `synced-actions/actions-stale` na instância corporativa de destino. Você deve criar a organização denominada `synced-actions` na sua empresa antes de executar o comando acima.
     * Se você omitir `:destination_owner/destination_repository`, a ferramenta usará o proprietário original e o nome do repositório para a sua empresa. Antes de executar o comando, você deve criar uma nova organização em sua empresa que corresponda ao nome da ação do proprietário. Considere usar uma organização central para armazenar as ações sincronizadas na sua empresa, uma vez que isso significa que você não precisará criar várias novas organizações se sincronizar ações de diferentes proprietários.
     * Você pode sincronizar várias ações substituindo o parâmetro `--repo-name` por `--repo-name-list` ou `--repo-name-list-file`. Para obter mais informações, consulte o README de [`actions-sync`](https://github.com/actions/actions-sync#actions-sync).
1. Depois que o repositório de ação for criado na sua empresa, as pessoas da sua empresa poderão usar o repositório de destino para fazer referência à ação nos fluxos de trabalho. Para o exemplo da ação mostrado acima:

   ```yaml
   uses: synced-actions/actions-stale@v1
   ```

   Para obter mais informações, consulte "[Sintaxe do fluxo de trabalho para o GitHub Actions](/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsuses)".