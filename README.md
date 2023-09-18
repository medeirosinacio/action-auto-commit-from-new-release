# GitHub Actions Example: Commit New Release

Este repositório é um exemplo de como criar uma `github action` para automatizar a tarefa de commitar uma nova release,
copiando o
nome e o corpo da release no arquivo de [CHANGELOG.md](CHANGELOG.md) do repositório. Isso pode servir para manter um
registro das versões
e atualizações do seu projeto de forma automática.

A ação é acionada sempre que uma nova release é publicada no repositório. A seguir, você encontrará uma explicação do
fluxo de trabalho e dos passos executados pela ação.

```yaml
name: Commit new Release

on:
  release:
    types:
      - 'published'
    tags:
      - '*.*'

jobs:
  commit:
    name: Commit Release
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          ref: main
          fetch-depth: 0
      - name: Commit new release
        run: |
          { printf "### Release ${{ github.event.release.tag_name }} \n\n${{ github.event.release.body }}\n\n"; cat CHANGELOG.md; } > tmp.txt && mv tmp.txt CHANGELOG.md
      - name: Push new release
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_user_name: github-actions-dump-bot
          commit_user_email: github-actions-dump-bot@dump.tec.br
          commit_author: Dump Bot <github-actions-dump-bot@dump.tec.br>
          commit_message: release ${{ github.event.release.tag_name }}
      - name: Create and push backup branch
        run: |
          backup_branch="release/${{ github.event.release.tag_name }}"
          git checkout -b "$backup_branch"
          git push origin "$backup_branch"
```

Neste fluxo de trabalho:

- A ação é acionada quando uma nova release é publicada, independentemente da tag da release.
- Ela é executada em uma máquina virtual Ubuntu.
- Os passos incluem:
    - Checkout: Isso verifica o código-fonte do repositório.
    - Commit new release: Este passo atualiza o arquivo de changelog do repositório com as informações da release. Ele
      adiciona uma entrada no formato markdown com o nome e o corpo da release.
    - Push new release: Este passo efetua o commit das mudanças no arquivo de changelog e as envia para o repositório.
    - Create and push backup branch: Este passo cria uma branch de backup para a release atual e a envia para o
      repositório remoto. A branch de backup é nomeada com base na tag da release.

Isso permite que você mantenha um registro organizado de todas as suas releases no arquivo de changelog e tenha uma
branch de backup correspondente para cada release publicada.
