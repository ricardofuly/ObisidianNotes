

![[Git-Icon-1788C.webp]]
# Comandos

`git —help` - Comando de Ajuda.

`git init` - para iniciar um repositório git.

> `git init --bare` - para tornar o repositório como remoto (servidor)

`git status` - para verificar o status do repositório.

```livescript
git config --local user.name "Seu nome aqui"
git config --local user.email "seu@email.aqui"
```

> Estes códigos acima são necessários para configurar as sua conta do github.

`git add .` - Para adicionar todos os arquivos ao git.

`git commit -m "mensagem do commit"` - Para adicionar um comentário no commit do seu repositório.

`git log` - Para verificar o log de alterações do repositório.

> —online - para exibir o último log, -p - para verificar as alterações

`git config` - Para verificar ou editar as configurações.

> —local - para mudar as configurações para o repositório atual, —global - para mudar as configurações para todos os repositórios.

[Comandos para o](https://devhints.io/git-log) `git log`

> `git log —online` - Mostra o Log dos commits.

`git remote add local` - Para adicionar uma pasta de um servidor

> `git remote add nome-repositorio caminho/para/o/repositorio`

`git clone` - para clonar um repositório.

> `git clone` URL nome do projeto

> `git clone -b branch link do projeto`

`git push` - Para enviar os dados para o repositório.

> `git push local master`

> `git push --set-upstream develop ricardo` - fazendo o push para a branch develop pela branch ricardo

`git pull` - Para buscar os dados do repositório

> `git push local master`

`git diff` - Exibe as modificações que o arquivo sofreu.

> `git diff —estado do arquivo` - estados —cached, —staged

`git checkout` - Navega entre as branchs

> `git checkout -b nome da branch` - Cria e muda para a branch criada.

![[WhatsApp Image 2023-02-14 at 14.38.43.jpeg]]

# Gerando Chave SSH

`ssh-keygen -t rsa -C "meu-email@email.com"` - Para gerar a chave SSH RSA.

`cat ~/.ssh/id_rsa.pub` - para exibir a chave SSH

# Clonando Repositório com os Submódulos

`—recursive` - Para clonar com os submódulos.

`git clone —recursive git.com/repo` - Exemplo.

# Fluxo de Trabalho

`git status` - Verifica se tem arquivos a serem commitados.
`git remote update` - Atualiza sua branch local.
`git checkout origin/master` - muda para a branch master.
`git submodule update` - atualiza os submodulos do projeto (Plugins ou Ferramentas).
`git checkout -b nome-da-sua-branch` - cria e muda para a branch que acabou de ser criada.
`git status` - Para garantir que não esqueceu nenhum arquivo para commitar.

# Enviando os Arquivos para o repositório

`git status` - Verifica se tem arquivos a serem commitados.
`git add caminho/dos/arquivos/para.enviar` - Adiciona os arquivo ao seu commit.
`git restore caminho/dos/arquivos/para.enviar` - Remove e reseta os arquivos que você não quer commitar.
`git status` - Verifica se tem arquivos a serem commitados.
`git commit -m "Mensagem do seu PR"` - Adiciona uma mensagem ao seu commit.
`git push origin nome-da-sua-branch` - Envia os dados do commit para a branch remota.
Agora só dar um CTRL + Click Esquerdo no link que está no chat para abrir a pagina do git e enviar o seu PR (Pull Request).


# Resolvendo Conflitos

`git remote update` - Atualiza a sua branch.
`git rebase origin/master` - recebe atualizações que estão na sua branch remota
`git status` - verifica o estado dos arquivos.
`git checkout --ours ou --theirs` **--ours** se for manter o da master, **--thiers** .se for alterações da sua branch seleciona qual versão você vai manter.
`git add /NomePasta/NomeArquivo.formato` - Adiciona os arquivos para ser commitados.
`git rebase --continue` - Da continuidade com o rebase.
`:wq` (Abrindo um terminal VIM digite esse comando pra prosseguir) - encerra o terminal VIM.
`git push -f origin sua-branch` - Envia de forma forçada para a sua branch remota.