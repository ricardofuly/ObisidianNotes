## UFUNCTION Meta Specifiers

### DisplayName

Altera o nome exibido no Blueprint para a função.

`UFUNCTION(BlueprintCallable, Category = "Exemplo", meta = (DisplayName = "Função Customizada")) void FuncaoExemplo();`

### Keywords

Define palavras-chave para facilitar a busca da função no editor.

`UFUNCTION(BlueprintCallable, Category = "Exemplo", meta = (Keywords = "exemplo, custom, função")) void FuncaoComKeywords();`

### ToolTip

Fornece uma descrição que aparece como dica de ferramenta no editor.

`UFUNCTION(BlueprintCallable, Category = "Exemplo", meta = (ToolTip = "Esta função realiza uma operação importante")) void FuncaoComTooltip();`

### DeprecatedFunction

Marca a função como obsoleta e recomenda uma alternativa (usualmente usada juntamente com a mensagem de depreciação).

`UFUNCTION(BlueprintCallable, Category = "Exemplo", meta = (DeprecatedFunction, DeprecationMessage = "Use NovaFuncao em vez desta função")) void FuncaoObsoleta();`

### CompactNodeTitle

Exibe um título mais curto no nó do Blueprint, deixando a visualização mais compacta.

`UFUNCTION(BlueprintCallable, Category = "Exemplo", meta = (CompactNodeTitle = "Exec")) void ExecFuncao();`

---

## UPROPERTY Meta Specifiers

### ClampMin e ClampMax

Restringe os valores que podem ser atribuídos à propriedade no editor.

`UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuração", meta = (ClampMin = "0.0", ClampMax = "1.0")) float ValorClamp;`

### DisplayName

Modifica o nome exibido no editor para a propriedade.

`UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuração", meta = (DisplayName = "Número Especial")) int32 NumeroEspecial;`

### EditCondition

Permite que a propriedade seja editada somente se uma condição (geralmente outra variável booleana) for verdadeira.

`UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuração", meta = (EditCondition = "bPodeEditar")) bool bPodeEditar;  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuração", meta = (EditCondition = "bPodeEditar")) float ValorEditavel;`

### BindWidget

Usado em classes derivadas de UUserWidget para associar uma variável C++ a um widget do Blueprint.

`UPROPERTY(meta = (BindWidget)) class UTextBlock* MeuTexto;`

### AdvancedDisplay

Marca a propriedade como avançada, fazendo com que ela só seja exibida quando o modo avançado estiver ativado no editor.

`UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuração", meta = (AdvancedDisplay)) int32 ValorAvancado;`

# if WITH EDITOR

### Sobrescrevendo Funções Específicas do Editor

Um exemplo clássico é sobrescrever funções que são chamadas somente no Editor, como `PostEditChangeProperty`. Esse método é invocado sempre que uma propriedade do objeto é modificada através do editor.

`#if WITH_EDITOR virtual void PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent) override {     Super::PostEditChangeProperty(PropertyChangedEvent);      // Código específico para o Editor, como atualizações visuais ou validação de dados.     UE_LOG(LogTemp, Warning, TEXT("Uma propriedade foi modificada no Editor!")); } #endif`

Nesse exemplo, o método `PostEditChangeProperty` é definido somente quando a flag `WITH_EDITOR` está ativa, evitando que esse código seja compilado para builds finais.

---

### Código Condicional Dentro de Métodos

Você pode usar `#if WITH_EDITOR` dentro de métodos para executar determinadas operações somente durante o desenvolvimento, como logs ou ajustes que não são necessários no jogo final.

`void AMyActor::AtualizarDados() {     // Código comum para todas as builds     // ...  #if WITH_EDITOR     // Executa código específico do Editor, como debug ou validação extra     UE_LOG(LogTemp, Warning, TEXT("Atualização de dados no Editor.")); #endif      // Continuação do código comum     // ... }`

Neste exemplo, a mensagem de log só será compilada e executada quando o código estiver rodando dentro do Editor, permitindo testes e validações sem afetar o produto final.

---

### Uso em Componentes ou Classes Específicas

Às vezes, pode ser necessário incluir variáveis ou métodos que só façam sentido durante o desenvolvimento ou design dos níveis. Por exemplo, você pode ter uma função para desenhar gizmos no Editor:

`#if WITH_EDITOR virtual void DrawDebugVisualizations() {     // Código para desenhar visualizações (gizmos) no Editor.     // Isso pode incluir chamadas para funções como DrawDebugLine, DrawDebugSphere, etc.     DrawDebugSphere(GetWorld(), GetActorLocation(), 50.0f, 12, FColor::Green, false, -1, 0, 2); } #endif`

Essa função pode ser chamada somente no Editor para ajudar na visualização de colisões, áreas de efeito ou outros dados de depuração.

---

### Considerações Gerais

- **Manutenção de Código:** Utilizar `#if WITH_EDITOR` ajuda a manter o código organizado e evita que dependências específicas do Editor sejam compiladas para as builds finais, garantindo melhor desempenho e evitando erros de referência a módulos que não existem na versão final.
- **Testes e Debug:** Esse recurso é especialmente útil durante o desenvolvimento para incluir logs, verificações e outras ferramentas que auxiliam na criação e depuração do projeto sem impactar a performance do jogo final.