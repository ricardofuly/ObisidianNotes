

![[Git-Icon-1788C.webp]]


## Comandos de Ajuda e Configuração

- **Ajuda do Git:**
    
    - `git --help`  
        Exibe a ajuda e documentação dos comandos Git.
- **Configuração de Usuário:**  
    Configure seu nome e e-mail para que os commits sejam atribuídos corretamente.
    
    `git config --local user.name "Seu nome aqui" git config --local user.email "seu@email.aqui"`
    
    > _Observação:_ Essas configurações podem ser definidas de forma local (somente para o repositório atual) ou global (para todos os repositórios) usando a opção `--global`.
    
- **Verificar/Editar Configurações:**
    
    - `git config`  
        Exibe ou permite editar as configurações do Git.
        
        > Use `--local` para o repositório atual ou `--global` para todas as instâncias.
        

---

## Inicializando Repositórios

- **Criar um novo repositório:**
    
    - `git init`  
        Inicializa um novo repositório Git no diretório atual.
- **Criar um repositório bare (remoto/servidor):**
    
    - `git init --bare`  
        Inicializa um repositório sem área de trabalho (ideal para servidores remotos).

---

## Controle de Alterações (Staging e Commit)

- **Verificar o status dos arquivos:**
    
    - `git status`  
        Mostra o estado atual do repositório, indicando arquivos modificados, adicionados ou não monitorados.
- **Adicionar arquivos para commit:**
    
    - `git add .`  
        Adiciona todos os arquivos e mudanças ao _staging area_.
- **Registrar as alterações com uma mensagem:**
    
    - `git commit -m "mensagem do commit"`  
        Cria um commit com as alterações adicionadas e inclui uma mensagem descritiva.

---

## Histórico e Diferenças

- **Histórico de commits:**
    
    - `git log`  
        Exibe o histórico dos commits do repositório.
        
        > Dicas:
        > 
        > - `git log --oneline` exibe um log resumido com cada commit em uma única linha.
        > - `git log -p` mostra as diferenças (patches) de cada commit.
        > 
        > Para mais opções, veja [Devhints Git Log](https://devhints.io/git-log).
        
- **Visualizar diferenças entre estados:**
    
    - `git diff`  
        Mostra as alterações entre o estado atual dos arquivos e o último commit.
        
        > Dicas:
        > 
        > - Use `git diff --cached` ou `git diff --staged` para ver as diferenças dos arquivos já adicionados ao _staging area_.
        

---

## Trabalhando com Branches

- **Navegar entre branches e criar uma nova branch:**
    - `git checkout`  
        Muda de branch.
        
        > Exemplo para criar e mudar para uma nova branch:  
        > `git checkout -b nome-da-branch`
        

---

## Repositórios Remotos e Clonagem

- **Adicionar um repositório remoto:**
    
    - `git remote add nome-repositorio caminho/para/o/repositorio`  
        Associa um repositório remoto ao seu projeto com o nome especificado.
- **Clonar um repositório:**
    
    - `git clone URL nome-do-projeto`  
        Clona o repositório do URL informado para uma nova pasta com o nome especificado.
    - **Clonando uma branch específica:**
        - `git clone -b nome-da-branch URL`  
            Clona o repositório e já muda para a branch desejada.

---

## Enviando e Recebendo Alterações (Push e Pull)

- **Enviar commits para o repositório remoto:**
    
    - `git push`  
        Transfere as alterações locais para o repositório remoto.
        
        > Exemplos:
        > 
        > - `git push local master`  
        >     Envia a branch master para o remoto nomeado "local".
        > - `git push --set-upstream develop ricardo`  
        >     Faz o push da branch develop, configurando o upstream para a branch "ricardo".
        
- **Buscar e integrar alterações do repositório remoto:**
    
    - `git pull`  
        Atualiza o repositório local com as alterações existentes no repositório remoto.
        
        > _Observação:_ No exemplo original, havia uma referência incorreta para `git push` em vez de `git pull`.

![[WhatsApp Image 2023-02-14 at 14.38.43.jpeg]]

### Gerando e Gerenciando Chaves SSH

### Gerando a Chave SSH Padrão

- **Gerar chave SSH RSA (padrão):**
        
    `ssh-keygen -t rsa -C "meu-email@email.com"`
    
    _Gera um par de chaves (pública e privada) utilizando o algoritmo RSA e adiciona um comentário (seu e-mail) para identificação._
    
- **Exibir a chave pública:**
        
    `cat ~/.ssh/id_rsa.pub`
    
    _Mostra o conteúdo da chave pública gerada, que pode ser copiada para plataformas como GitHub ou GitLab._
    

---

### Criando uma Nova Chave SSH (com configurações avançadas)

- **Listar arquivos do diretório SSH:**
    
    `ls -l ~/.ssh/`
    
    _Exibe os arquivos presentes em `~/.ssh/` com detalhes, ajudando a identificar chaves já existentes._
    
- **Gerar nova chave SSH com parâmetros customizados:**
        
    `ssh-keygen -t rsa -b 4096 -C "seu-email@example.com" -f ~/.ssh/minha-nova-chave`
    
    _Gera um par de chaves RSA com 4096 bits, adicionando um comentário e salvando em um arquivo específico (`minha-nova-chave`), criando também o arquivo público (`minha-nova-chave.pub`)._
    
- **Iniciar o agente SSH:**
    
    `eval "$(ssh-agent -s)"`
    
    _Inicia o processo do agente SSH, que gerencia suas chaves e facilita a autenticação sem precisar inserir a senha repetidamente._
    
- **Adicionar chaves ao agente SSH:**
    
    - Adicionando a chave padrão:
                
        `ssh-add ~/.ssh/id_rsa`
        
    - Adicionando a nova chave criada:
                
        `ssh-add ~/.ssh/minha-nova-chave`
        
    
    _Ambos os comandos adicionam suas respectivas chaves privadas ao agente SSH para uso em conexões seguras._
    
- **Listar chaves carregadas no agente:**
        
    `ssh-add -l`
    
    _Exibe a lista de chaves atualmente gerenciadas pelo agente._
    
- **Exibir a nova chave pública:**
        
    `cat ~/.ssh/minha-nova-chave.pub`
    
    _Mostra o conteúdo da nova chave pública, que deve ser adicionada nos serviços que exigem autenticação SSH._
    

---

## Clonando Repositórios com Submódulos

- **Clonagem com submódulos (opção `--recursive`):**
        
    `git clone git.com/repo --recursive`
    
    _Clona o repositório incluindo todos os submódulos configurados._

> _Observação:_ Algumas vezes, o parâmetro pode aparecer como `--recurse-submodules`, que tem a mesma função.

---

## Fluxo de Trabalho com Git

### Atualização e Gerenciamento do Repositório

- **Verificar o status dos arquivos:**
        
    `git status`
    
    _Mostra quais arquivos foram modificados, adicionados ou estão pendentes de commit._
    
- **Atualizar o repositório local com o remoto:**
        
    `git remote update`
    
    _Atualiza as referências do repositório remoto._
    
- **Navegar para a branch master do remoto:**
        
    `git checkout origin/master`
    
    _Muda para a branch master do repositório remoto (somente leitura)._
    
- **Atualizar submódulos do projeto:**
        
    `git submodule update`
    
    _Sincroniza os submódulos com as configurações definidas no repositório principal._
    
- **Criar e mudar para uma nova branch:**
        
    `git checkout -b nome-da-sua-branch`
    
    _Cria uma nova branch e muda para ela, preparando o ambiente para novos commits._
    
- **Verificar novamente o status:**
        
    `git status`
    
    _Confirma se não há alterações pendentes ou arquivos esquecidos._
    

---

## Enviando Arquivos para o Repositório

### Preparação e Envio de Alterações

- **Verificar o status antes de enviar:**
        
    `git status`
    
    _Confirma quais arquivos estão prontos para commit._
    
- **Adicionar arquivos específicos ao commit:**
        
    `git add caminho/dos/arquivos/para.enviar`
    
    _Prepara os arquivos selecionados para o próximo commit._
    
- **Restaurar/descartar arquivos que não serão commitados:**
    
    `git restore caminho/dos/arquivos/para.enviar`
    
    _Remove alterações de arquivos que não devem ser incluídos no commit._
    
- **Confirmar o status novamente:**
        
    `git status`
    
    _Garante que somente os arquivos desejados estejam prontos para commit._
    
- **Criar um commit com mensagem:**
        
    `git commit -m "Mensagem do seu PR"`
    
    _Registra as alterações com uma mensagem descritiva (útil para Pull Requests)._
    
- **Enviar os commits para o repositório remoto:**
    
    `git push origin nome-da-sua-branch`
    
    _Envia as alterações para a branch remota correspondente._
    

> _Dica:_ Após o push, abra o link gerado (geralmente enviado via chat ou interface web) para criar o Pull Request (PR).

---

## Resolvendo Conflitos

### Processo de Resolução de Conflitos durante um Rebase

- **Atualizar referências do repositório remoto:**
    
    `git remote update`
    
    _Garante que as informações estejam atualizadas antes de iniciar o rebase._
    
- **Realizar o rebase com a branch master:**
        
    `git rebase origin/master`
    
    _Traz as alterações da branch master para a sua branch atual._
    
- **Verificar o status dos arquivos:**
    
    `git status`
    
    _Identifica quais arquivos estão em conflito._
    
- **Selecionar qual versão manter:**
    
    `git checkout --ours caminho/para/arquivo git checkout --theirs caminho/para/arquivo`
    
    _Utilize `--ours` para manter a versão da branch base (master) ou `--theirs` para manter as alterações da sua branch._
    
- **Adicionar os arquivos resolvidos:**
    
    `git add /NomePasta/NomeArquivo.formato`
    
    _Marca os conflitos como resolvidos para que o rebase possa continuar._
    
- **Continuar o rebase:**
    
    `git rebase --continue`
    
    _Prossegue com o processo de rebase após resolver os conflitos._
    
- **Encerrar o editor de texto (no VIM):**
        
    `:wq`
    
    _No editor VIM, esse comando salva as alterações e fecha o editor._
    
- **Forçar o envio para o repositório remoto:**
        
    `git push -f origin sua-branch`
    
    _Realiza um push forçado para atualizar a branch remota com as alterações rebaseadas._
    

---

## Gerenciando Submódulos

### Adicionar e Atualizar Submódulos

- **Adicionar um submódulo ao repositório:**
        
    `git submodule add <URL-do-repositório> <nome-da-pasta/nome-do-plugin>`
    
    _Inclui um submódulo no seu projeto, organizando código ou ferramentas externas em uma pasta específica._
    
- **Inicializar e atualizar submódulos:**
        
    `git submodule update --init --recursive`
    
    _Garante que todos os submódulos sejam baixados e sincronizados com o repositório principal._
    
- **Clonar um repositório com submódulos automaticamente:**
    
    `git clone --recurse-submodules <URL-do-repositório>`
    
    _Clona o repositório e seus submódulos em uma única etapa._
    
- **Atualizar submódulos após mudanças no repositório principal:**
        
    `git submodule update --remote --merge`
    
    _Atualiza os submódulos para as últimas versões disponíveis, integrando eventuais mudanças._
    

### Remover um Submódulo

- **Desativar o submódulo:**
        
    `git submodule deinit -f <caminho-do-submódulo>`
    
- **Remover os arquivos fisicamente:**
        
    `rm -rf <caminho-do-submódulo>`
    
- **Remover o submódulo do Git:**
        
    `git rm -f <caminho-do-submódulo>`
    
    _Esses passos removem completamente o submódulo do seu projeto._