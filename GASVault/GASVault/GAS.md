
### Ability System Component

Inicializar o Ability System Component como os dados fornecidos
```
UFUNCTION(BlueprintCallable)  
void InitializeAbilitySystemData(const FAbilitySystemInitializationData& InitializationData, AActor* InOwningActor, AActor* InAvatarActor);
```

criar um wrapper do "GetOrCreateAttributeSubobject" function, Retorna um Attribute set ou retorna um se não encontrado
```
const UAttributeSet* GetOrCreateAttributeSet(const TSubclassOf<UAttributeSet>& InAttributeSet);
```

Uma bool para inicializar o ability system
```
bool AbilitySystemDataInitialized = false;
```

Implementação do constructor

```
ReplicationMode = EGameplayEffectReplicationMode::Mixed;
```

```
SetIsReplicatedByDefault(true);
```

Implementação do GetOrCreateAttributeSet function
```
return GetOrCreateAttributeSubobject(InAttributeSet);
```

Implementação do InitializeAbilitySystemData function
```
if (AbilitySystemDataInitialized)  
{  
    return;  
}  
  
AbilitySystemDataInitialized = true;  
  
// Set the Owning Actor and Avatar Actor. (Used throughout the Gameplay Ability System to get references etc.)  
InitAbilityActorInfo(InOwningActor, InAvatarActor);  
  
// Check to see if we have authority. (Attribute Sets / Attribute Base Values / Gameplay Abilities / Gameplay Effects should only be added -or- set on authority and will be replicated to the client automatically.)  
if (GetOwnerActor()->HasAuthority())  
{  
    // Grant Attribute Sets if the array isn't empty.  
    if (!InitializationData.AttributeSets.IsEmpty())  
    {       for (const TSubclassOf<UAttributeSet> AttributeSetClass : InitializationData.AttributeSets)  
       {          GetOrCreateAttributeSet(AttributeSetClass);  
       }    }  
    // Set base attribute values if the map isn't empty.  
    if (!InitializationData.AttributeBaseValues.IsEmpty())  
    {       for (const TTuple<FGameplayAttribute, float>& AttributeBaseValue : InitializationData.AttributeBaseValues)  
       {          if (HasAttributeSetForAttribute(AttributeBaseValue.Key))  
          {             SetNumericAttributeBase(AttributeBaseValue.Key, AttributeBaseValue.Value);  
          }       }    }  
    // Grant Gameplay Abilities if the array isn't empty.  
    if (!InitializationData.GameplayAbilities.IsEmpty())  
    {       for (const TSubclassOf<UGameplayAbility> GameplayAbility : InitializationData.GameplayAbilities)  
       {          FGameplayAbilitySpec AbilitySpec = FGameplayAbilitySpec(GameplayAbility, 1, INDEX_NONE, this);  
          GiveAbility(AbilitySpec);  
       }    }  
    // Apply Gameplay Effects if the array isn't empty.  
    if (!InitializationData.GameplayEffects.IsEmpty())  
    {       for (const TSubclassOf<UGameplayEffect>& GameplayEffect : InitializationData.GameplayEffects)  
       {          if (IsValid(GameplayEffect))  
          {             FGameplayEffectContextHandle EffectContextHandle = MakeEffectContext();  
             EffectContextHandle.AddSourceObject(this);  
  
             if (FGameplayEffectSpecHandle GameplayEffectSpecHandle = MakeOutgoingSpec(GameplayEffect, 1, EffectContextHandle); GameplayEffectSpecHandle.IsValid())  
             {                ApplyGameplayEffectSpecToTarget(*GameplayEffectSpecHandle.Data.Get(), this);  
             }          }       }    }}  
  
// Apply the Gameplay Tag container as loose Gameplay Tags. (These are not replicated by default and should be applied on both server and client respectively.)  
if (!InitializationData.GameplayTags.IsEmpty())  
{  
    AddLooseGameplayTags(InitializationData.GameplayTags);  
}
```


### PlayerState

Forward Declaration do Ability System
```
class UYOURGAMEAbilitySystemComponent;
```


Incluir Ability System Interface
```
#include "AbilitySystemInterface.h"
```

Criar o Componente
```
UPROPERTY(BlueprintReadOnly, VisibleAnywhere, Category="Ability System");
UAbilitySystemComponent* AbilitySystemComponent;
```

Implementar a função de Get da interface
```
virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;
```

No .cpp
Definir a frequência de update 
```
NetUpdateFrequency = 100.0f;
```

Criar o Objeto do Ability System
```
AbilitySystemComponent = CreateDefaultSubobject<UYOURGAMEAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
```

Implementar a Função GetAbilitySystemComponent() retornando o AbilitySystemComponent
```
return AbilitySystemComponent
```


### Attribute Set 

Criando a Classe Base

Definindo as Macros de Support
```
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \  
       GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \  
       GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \  
       GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \  
       GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

dentro da protected section
```
void AdjustAttributeForMaxChange(const FGameplayAttributeData& AffectedAttribute, const FGameplayAttributeData& MaxAttribute, float NewMaxValue, const FGameplayAttribute& AffectedAttributeProperty) const;
```

Implementação da função
```
UAbilitySystemComponent* AbilitySystemComponent = GetOwningAbilitySystemComponent();  
  
if (const float CurrentMaxValue = MaxAttribute.GetCurrentValue(); !FMath::IsNearlyEqual(CurrentMaxValue, NewMaxValue) && AbilitySystemComponent)  
{  
    // Change current value to maintain the Current Value / Maximum Value percentage.  
    const float CurrentValue = AffectedAttribute.GetCurrentValue();  
    const float NewDelta = CurrentMaxValue > 0.f ? CurrentValue * NewMaxValue / CurrentMaxValue - CurrentValue : NewMaxValue;  
  
    AbilitySystemComponent->ApplyModToAttributeUnsafe(AffectedAttributeProperty, EGameplayModOp::Additive, NewDelta);  
}
```

##### Criando o Atributo de Vida

Crie uma child class do Attributo Base

Declarando as funções para sobre escrever os atributos
```
// Attribute Set Overrides.  
virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;  
virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;
```

Declarando a função de replicação
```
// Set Attributes to replicate.  
virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
```

Criando os Attribute Data

Incomming Damage
```
// Used to create a local copy of Damage which is then subtracted from Current Health.  
UPROPERTY(BlueprintReadOnly, Category = "Health Attribute Set", meta = (HideFromLevelInfos))  
FGameplayAttributeData Damage;  
ATTRIBUTE_ACCESSORS(UHealthAttributeSet, Damage)
```

Healing
```
// Used to create a local copy of Healing which is then added to Current Health.  
UPROPERTY(BlueprintReadOnly, Category = "Health Attribute Set", meta = (HideFromLevelInfos))  
FGameplayAttributeData Healing;  
ATTRIBUTE_ACCESSORS(UHealthAttributeSet, Healing)
```

Current Health
```
// Holds the current value for Health.  
UPROPERTY(BlueprintReadOnly, Category = "Health Attribute Set", ReplicatedUsing = OnRep_CurrentHealth)  
FGameplayAttributeData CurrentHealth;  
ATTRIBUTE_ACCESSORS(UHealthAttributeSet, CurrentHealth)
```

Max Health
```
// Holds the value for Maximum Health.  
UPROPERTY(BlueprintReadOnly, Category = "Health Attribute Set", ReplicatedUsing = OnRep_MaximumHealth)  
FGameplayAttributeData MaximumHealth;  
ATTRIBUTE_ACCESSORS(UHealthAttributeSet, MaximumHealth)
```

Health Regeneration
```
// Holds the value for Health Regeneration.  
UPROPERTY(BlueprintReadOnly, Category = "Health Attribute Set", ReplicatedUsing = OnRep_HealthRegeneration)  
FGameplayAttributeData HealthRegeneration;  
ATTRIBUTE_ACCESSORS(UHealthAttributeSet, HealthRegeneration)
```

Na Protected Section
Crie as Funções de Rep_Notify da Vida

Current Health
```
UFUNCTION()  
virtual void OnRep_CurrentHealth(const FGameplayAttributeData& OldValue);
```

Max Health
```
UFUNCTION()  
virtual void OnRep_MaximumHealth(const FGameplayAttributeData& OldValue);
```

Health Regeneration
```
UFUNCTION()  
virtual void OnRep_HealthRegeneration(const FGameplayAttributeData& OldValue);
```

## No .cpp

Inicializando as variáveis no construtor
```
MaximumHealth = 0.0f;  
CurrentHealth = 0.0f;  
HealthRegeneration = 0.0f;
```

Implementando a Função de replicação
```
DOREPLIFETIME_CONDITION_NOTIFY(UHealthAttributeSet, CurrentHealth, COND_None, REPNOTIFY_Always);  
DOREPLIFETIME_CONDITION_NOTIFY(UHealthAttributeSet, MaximumHealth, COND_None, REPNOTIFY_Always);  
DOREPLIFETIME_CONDITION_NOTIFY(UHealthAttributeSet, HealthRegeneration, COND_None, REPNOTIFY_Always);
```

Implementando a função PreAttributeChange
```
if (Attribute == GetMaximumHealthAttribute())  
{  
    AdjustAttributeForMaxChange(CurrentHealth, MaximumHealth, NewValue, GetCurrentHealthAttribute());  
}
```

Implementando a função PostGameplayEffectExecute
```
if (Data.EvaluatedData.Attribute == GetDamageAttribute())  
{  
    // Store a local copy of the amount of Damage done and clear the Damage attribute.  
    const float LocalDamageDone = GetDamage();  
  
    SetDamage(0.f);  
  
    if (LocalDamageDone > 0.0f)  
    {       // Apply the Health change and then clamp it.  
       const float NewHealth = GetCurrentHealth() - LocalDamageDone;  
  
       SetCurrentHealth(FMath::Clamp(NewHealth, 0.0f, GetMaximumHealth()));  
    }}  
  
else if (Data.EvaluatedData.Attribute == GetHealingAttribute())  
{  
    // Store a local copy of the amount of Healing done and clear the Healing attribute.  
    const float LocalHealingDone = GetHealing();  
  
    SetHealing(0.f);  
  
    if (LocalHealingDone > 0.0f)  
    {       // Apply the Health change and then clamp it.  
       const float NewHealth = GetCurrentHealth() + LocalHealingDone;  
  
       SetCurrentHealth(FMath::Clamp(NewHealth, 0.0f, GetMaximumHealth()));  
    }}  
  
else if (Data.EvaluatedData.Attribute == GetCurrentHealthAttribute())  
{  
    SetCurrentHealth(FMath::Clamp(GetCurrentHealth(), 0.0f, GetMaximumHealth()));  
}  
  
else if (Data.EvaluatedData.Attribute == GetHealthRegenerationAttribute())  
{  
    SetHealthRegeneration(FMath::Clamp(GetHealthRegeneration(), 0.0f, GetMaximumHealth()));  
}
```

Implementando os Rep_Notify

**OnRep_CurrentHealth**
```
GAMEPLAYATTRIBUTE_REPNOTIFY(UHealthAttributeSet, CurrentHealth, OldValue);
```

**OnRep_MaxHealth**
```
GAMEPLAYATTRIBUTE_REPNOTIFY(UHealthAttributeSet, MaximumHealth, OldValue);
```

**OnRep_HealthRegeneration**
```
GAMEPLAYATTRIBUTE_REPNOTIFY(UHealthAttributeSet, HealthRegeneration, OldValue);
```


# Ability System Data

Forward Declare Attribute Set, Gameplay Ability e Gameplay Effect
```
class UAttributeSet;  
class UGameplayAbility;  
class UGameplayEffect;
```

Criando Struct de Inicialização  de Dados
```
USTRUCT(BlueprintType)  
struct FAbilitySystemInitializationData  
{  
    GENERATED_BODY()  
  
    // An array of Attribute Sets to create.  
    UPROPERTY(BlueprintReadOnly, EditAnywhere)  
    TArray<TSubclassOf<UAttributeSet>> AttributeSets;  
  
    // A map of Attributes / float used to set base values.  
    UPROPERTY(BlueprintReadOnly, EditAnywhere)  
    TMap<FGameplayAttribute, float> AttributeBaseValues;  
  
    // An Array of Gameplay Abilities to give.  
    UPROPERTY(BlueprintReadOnly, EditAnywhere)  
    TArray<TSubclassOf<UGameplayAbility>> GameplayAbilities;  
  
    // An array of Gameplay Effects to apply.  
    UPROPERTY(BlueprintReadOnly, EditAnywhere)  
    TArray<TSubclassOf<UGameplayEffect>> GameplayEffects;  
  
    // A container of GameplayTags to apply.  
    UPROPERTY(BlueprintReadOnly, EditAnywhere)  
    FGameplayTagContainer GameplayTags;  
};
```

Criando Enum de Tipo de valores para os atributos
```
UENUM(BlueprintType)  
enum class EAttributeSearchType : uint8  
{  
    // Returns the final value of the Attribute including all stateful Gameplay Effect modifiers.  
    FinalValue,  
  
    // Returns the base value of the Attribute. (Excludes duration based Gameplay Effect modifiers)  
    BaseValue,  
  
    // Returns the Final Value minus the Base Value.  
    BonusValue  
};
```


# Native Gameplay Tags

Para as Gameplay tags nativas precisamos apenas de uma classe vazia com os includes padrão da unreal
```
#pragma once  
  
#include "CoreMinimal.h"  
#include "Runtime/GameplayTags/Public/NativeGameplayTags.h"
```

agora podemos separa as categorias das tags por name spaces
```
namespace NativeGameplayTags  
{  
    namespace CharacterTags
```

para criar as tags utilizaremos uma macros própria para isso
```
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Character_Type_PC);  
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Character_Type_NPC);  
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Character_State_BlockHealthRegen);
```

agora precisaremos de outra macro, mas dessa vez no .cpp
```
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Character_Type_PC, "Character.Type.PlayerCharacter", "A Gameplay Tag applied to Characters that are controlled by a local player.")  
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Character_Type_NPC, "Character.Type.NonPlayerCharacter", "A Gameplay Tag applied to Characters that are AI controlled.")  
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Character_State_BlockHealthRegen, "Character.State.BlockHealthRegen", "A Gameplay Tag applied to a Character that is used in Gameplay Effects to block health regeneration.")
```

dentro dos respectivos name spaces.


# Gameplay Ability

Variável bool para ativar a habilidade imediatamente quando garantida
```
// Tells an ability to activate immediately when it's granted. (Useful for passive abilities and abilities forced on others)  
UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "Custom Gameplay Ability")  
bool ActivateAbilityOnGranted = false;
```

Uma função para retornar no avatar character
```
// Returns the "Avatar Character" associated with this Gameplay Ability.  
// Will return null if the Avatar Actor does not derive from Character.  
UFUNCTION(BlueprintCallable, BlueprintPure)  
ACharacter* GetAvatarCharacter() const { return AvatarCharacter.Get(); }
```

Variável para salvar o ponteiro do avatar character
```
// Keep a pointer to "Avatar Character" so we don't have to cast to Character in instanced abilities owned by a Character derived class.  
TWeakObjectPtr<ACharacter> AvatarCharacter = nullptr;
```

Função para inicializar a habilidade quando ela for inicializada 
```
// Think of this as "BeginPlay".  
// Add logic here that should run when the Ability is first initialized.  
virtual void OnAvatarSet(const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilitySpec& Spec) override;
```

### No .Cpp

Definir para que a habilidade seja instanciada por actor por padrão
```
// Sets the ability to default to Instanced Per Actor.  
InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
```

Implementação da função OnAvatarSet
```
Super::OnAvatarSet(ActorInfo, Spec);  
  
// Set the "Avatar Character" reference.  
AvatarCharacter = Cast<ACharacter>(ActorInfo->AvatarActor);  
    
// Try to Activate immediately if "Activate Ability On Granted" is true.  
if (ActivateAbilityOnGranted)  
{  
    ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle);  
}
```


# Function Library

Biblioteca de funções

Vamos fazer um forward declaration de algumas classes que usaremos aqui
```
class UAbilitySystemComponent;  
class UGameplayAbility;  
class UAttributeSet;  
  
enum class EAttributeSearchType : uint8;
```

Precisaremos do Include do AttributeSet também
```
#include "AttributeSet.h"
```

Função para pegar o  valor de um atributo de um actor
```
// Tries to find the Actor's Ability System Component using the IAbilitySystemInterface.  
// If the Ability System Component is found; attempts to find the value of the Attribute supplied.  
UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Ability System")  
static float GetAttributeValueFromActor(const AActor* Actor, const FGameplayAttribute Attribute, const EAttributeSearchType SearchType);
```

Função para pegar o  valor de um atributo do ability system
```
// Attempts to find the value of the Attribute supplied.  
UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Ability System")  
static float GetAttributeValueFromAbilitySystem(const UAbilitySystemComponent* AbilitySystemComponent, const FGameplayAttribute Attribute, const EAttributeSearchType SearchType);
```

Uma função privada para pegar o valor do atributo
```
static void GetAttributeValue(const UAbilitySystemComponent* AbilitySystemComponent, const FGameplayAttribute& Attribute, const EAttributeSearchType SearchType, float& ReturnValue);
```

## No .cpp

Implementando as funções

GetAttributeValueFromActor()
```
float ReturnValue = -1.0f;  
  
if (const UAbilitySystemComponent* AbilitySystemComponent = UAbilitySystemGlobals::GetAbilitySystemComponentFromActor(Actor))  
{  
    GetAttributeValue(AbilitySystemComponent, Attribute, SearchType, ReturnValue);  
}  
  
return ReturnValue;
```

GetAttributeValueFromAbilitySystem()
```
float ReturnValue = -1.0f;  
  
GetAttributeValue(AbilitySystemComponent, Attribute, SearchType, ReturnValue);  
  
return ReturnValue;
```

GetAttributeValue()
```
ReturnValue = -1.0f;  
  
if (!AbilitySystemComponent || !AbilitySystemComponent->HasAttributeSetForAttribute(Attribute))  
{  
    return;  
}  
  
switch (SearchType)  
{  
    case EAttributeSearchType::FinalValue :  
    {  
       ReturnValue = AbilitySystemComponent->GetNumericAttribute(Attribute);  
  
       return;  
    }  
case EAttributeSearchType::BaseValue :  
    {  
       ReturnValue = AbilitySystemComponent->GetNumericAttributeBase(Attribute);  
  
       return;  
    }  
case EAttributeSearchType::BonusValue :  
    {  
       ReturnValue = AbilitySystemComponent->GetNumericAttribute(Attribute) - AbilitySystemComponent->GetNumericAttributeBase(Attribute);  
    }}
```


# Ability Task

On Tick Event

```
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTickEventDelegate, float, DeltaTime);
```

```
UPROPERTY(BlueprintAssignable)  
FTickEventDelegate TickEventReceived;
```

```
UFUNCTION(BlueprintCallable, Meta = (HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "True"), Category = "Ability Tasks")  
static UAbilityTask_OnTickEvent* OnTickEvent(UGameplayAbility* OwningAbility, const FName TaskInstanceName);
```

Protected Section

```
virtual void TickTask(const float DeltaTime) override;  
  
void EventReceived(const float DeltaTime) const;  
  
virtual void OnDestroy(const bool bInOwnerFinished) override;
```

## No .cpp

Constructor
```
bTickingTask = true;  
bSimulatedTask = true;
```

OnTickEvent()
```
return NewAbilityTask<UAbilityTask_OnTickEvent>(OwningAbility, TaskInstanceName);
```

TickTask()
```
Super::TickTask(DeltaTime);  
  
EventReceived(DeltaTime);
```

EventReceived()
```
if (TickEventReceived.IsBound())  
{  
    TickEventReceived.Broadcast(DeltaTime);  
}
```

OnDestroy()
```
bTickingTask = false;  
  
Super::OnDestroy(bInOwnerFinished);
```


WaitEnhancedInputEvent()

```
#include "EnhancedInputComponent.h"
```

Forward Declaration
```
class UInputAction;
```

Public:
```
UPROPERTY(BlueprintAssignable)  
FEnhancedInputEventDelegate InputEventReceived;  
  
UFUNCTION(BlueprintCallable, meta = (HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"), Category = "Ability Tasks")  
static UAbilityTask_WaitEnhancedInputEvent* WaitEnhancedInputEvent(UGameplayAbility* OwningAbility, const FName TaskInstanceName, UInputAction* InputAction, const ETriggerEvent TriggerEventType, bool bShouldOnlyTriggerOnce = true);
```

Private:
```
TWeakObjectPtr<UEnhancedInputComponent> EnhancedInputComponent = nullptr;  
  
TWeakObjectPtr<UInputAction> InputAction = nullptr;  
  
ETriggerEvent EventType;  
  
bool bTriggerOnce;  
  
bool bHasBeenTriggered = false;  
  
virtual void Activate() override;  
  
void EventReceived(const FInputActionValue& Value);  
  
virtual void OnDestroy(const bool bInOwnerFinished) override;
```


## No .cpp

WaitEnhancedInputEvent()
```
UAbilityTask_WaitEnhancedInputEvent* AbilityTask = NewAbilityTask<UAbilityTask_WaitEnhancedInputEvent>(OwningAbility, TaskInstanceName);  
  
AbilityTask->InputAction = InputAction;  
AbilityTask->EventType = TriggerEventType;  
AbilityTask->bTriggerOnce = bShouldOnlyTriggerOnce;  
  
return AbilityTask;
```

Activate()
```
Super::Activate();  
  
if (!AbilitySystemComponent.Get() || !Ability || !InputAction.IsValid())  
{  
    return;  
}  
  
if (const APawn* AvatarPawn = Cast<APawn>(Ability->GetAvatarActorFromActorInfo()))  
{  
    if (const APlayerController* PlayerController = Cast<APlayerController>(AvatarPawn->GetController()))  
    {       EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerController->InputComponent);  
       if (IsValid(EnhancedInputComponent.Get()))  
       {          EnhancedInputComponent->BindAction(InputAction.Get(), EventType, this, &UAbilityTask_WaitEnhancedInputEvent::EventReceived);  
       }    }}
```

EventReceived()
```
if (bTriggerOnce && bHasBeenTriggered)  
{  
    return;  
}  
  
bHasBeenTriggered = true;  
  
InputEventReceived.Broadcast(Value);
```

OnDestroy()
```
if (IsValid(EnhancedInputComponent.Get()))  
{  
    EnhancedInputComponent->ClearBindingsForObject(this);  
}  
  
Super::OnDestroy(bInOwnerFinished);
```


# Subsystem

```
virtual void Initialize(FSubsystemCollectionBase& Collection) override;
```

## No .cpp

Initialize()
```
Super::Initialize(Collection);  
  
UAbilitySystemGlobals::Get().InitGlobalData();
```


# Widget

Public:
```
// Should this widget bind to Health Attribute Set events.  
// Note: Initialization will fail if the required Attribute Set is not found!  
UPROPERTY(BlueprintReadOnly, EditAnywhere)  
bool ListenForHealthAttributeSetChanges = true;  
  
// Should this widget bind to Stamina Attribute Set events.  
// Note: Initialization will fail if the required Attribute Set is not found!  
UPROPERTY(BlueprintReadOnly, EditAnywhere)  
bool ListenForStaminaAttributeSetChanges = true;  
  
// Called to initialize the User Widget and bind to Attribute change delegates  
// Can be called again to re-initialize the values  
UFUNCTION(BlueprintCallable, Category = "Ability System")  
bool InitializeAbilitySystemWidget(UAbilitySystemComponent* InOwnerAbilitySystemComponent);  
  
// Returns the Owner's Ability System Component.  UFUNCTION(BlueprintCallable, BlueprintPure)  
UAbilitySystemComponent* GetOwnerAbilitySystemComponent() const;  
  
// Event called when the Maximum Health attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_MaximumHealthChanged(const float NewValue, const float OldValue, const float NewPercentage);  
  
// Event called when the Current Health attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_CurrentHealthChanged(const float NewValue, const float OldValue, const float NewPercentage);  
  
// Event called when the Health Regeneration attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_HealthRegenerationChanged(const float NewValue, const float OldValue);  
  
// Event called when the  Maximum Stamina attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_MaximumStaminaChanged(const float NewValue, const float OldValue, const float NewPercentage);  
  
// Event called when the Current Stamina attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_CurrentStaminaChanged(const float NewValue, const float OldValue, const float NewPercentage);  
  
// Event called when the Stamina Regeneration attribute value changes.  
UFUNCTION(BlueprintImplementableEvent, Category = "Ability System")  
void On_StaminaRegenerationChanged(const float NewValue, const float OldValue);
```

Protected:
```
TWeakObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;  
  
FDelegateHandle MaximumHealthChangeDelegate;  
FDelegateHandle CurrentHealthChangeDelegate;  
FDelegateHandle HealthRegenerationChangeDelegate;  
FDelegateHandle MaximumStaminaChangeDelegate;  
FDelegateHandle CurrentStaminaChangeDelegate;  
FDelegateHandle StaminaRegenerationChangeDelegate;  
  
void MaximumHealthChanged(const FOnAttributeChangeData& Data);  
  
void CurrentHealthChanged(const FOnAttributeChangeData& Data);  
  
void HealthRegenerationChanged(const FOnAttributeChangeData& Data);  
  
void MaximumStaminaChanged(const FOnAttributeChangeData& Data);  
  
void CurrentStaminaChanged(const FOnAttributeChangeData& Data);  
  
void StaminaRegenerationChanged(const FOnAttributeChangeData& Data);  
  
static void ResetDelegateHandle(FDelegateHandle DelegateHandle, UAbilitySystemComponent* OldAbilitySystemComponent, const FGameplayAttribute& Attribute);
```

# No .cpp

GetOwnerAbilitySystemComponent()
```
return AbilitySystemComponent.Get();
```

InitializeAbilitySystemWidget()
```
UAbilitySystemComponent* OldAbilitySystemComponent = AbilitySystemComponent.Get();  
  
AbilitySystemComponent = InOwnerAbilitySystemComponent;  
  
// The Ability System Component is invalid. Stop here and return false. if (!GetOwnerAbilitySystemComponent())  
{  
    return false;  
}  
  
// Reset any old Attribute Change Delegates if they are still bound.  
if (IsValid(OldAbilitySystemComponent))  
{  
    ResetDelegateHandle(MaximumHealthChangeDelegate, OldAbilitySystemComponent, UHealthAttributeSet::GetMaximumHealthAttribute());  
    ResetDelegateHandle(CurrentHealthChangeDelegate, OldAbilitySystemComponent, UHealthAttributeSet::GetCurrentHealthAttribute());  
    ResetDelegateHandle(HealthRegenerationChangeDelegate, OldAbilitySystemComponent, UHealthAttributeSet::GetHealthRegenerationAttribute());  
    ResetDelegateHandle(MaximumStaminaChangeDelegate, OldAbilitySystemComponent, UStaminaAttributeSet::GetMaximumStaminaAttribute());  
    ResetDelegateHandle(CurrentStaminaChangeDelegate, OldAbilitySystemComponent, UStaminaAttributeSet::GetCurrentStaminaAttribute());  
    ResetDelegateHandle(StaminaRegenerationChangeDelegate, OldAbilitySystemComponent, UStaminaAttributeSet::GetStaminaRegenerationAttribute());  
}  
  
// Bind Health attribute delegates if the Ability System Component has the required Attribute Set -and- we are listening for Health attributes.  
if (ListenForHealthAttributeSetChanges)  
{  
    if (AbilitySystemComponent->HasAttributeSetForAttribute(UHealthAttributeSet::GetMaximumHealthAttribute()))  
    {       MaximumHealthChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UHealthAttributeSet::GetMaximumHealthAttribute()).AddUObject(this, &UAbilitySystemWidget::MaximumHealthChanged);  
       CurrentHealthChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UHealthAttributeSet::GetCurrentHealthAttribute()).AddUObject(this, &UAbilitySystemWidget::CurrentHealthChanged);  
       HealthRegenerationChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UHealthAttributeSet::GetHealthRegenerationAttribute()).AddUObject(this, &UAbilitySystemWidget::HealthRegenerationChanged);  
  
       const float MaxHealth = AbilitySystemComponent->GetNumericAttribute(UHealthAttributeSet::GetMaximumHealthAttribute());  
       const float CurrentHealth = AbilitySystemComponent->GetNumericAttribute(UHealthAttributeSet::GetCurrentHealthAttribute());  
          // Call the Blueprint Events to initialize the values.  
       On_MaximumHealthChanged(MaxHealth, 0.0f, CurrentHealth / MaxHealth);  
       On_CurrentHealthChanged(CurrentHealth, 0.0f, CurrentHealth / MaxHealth);  
       On_HealthRegenerationChanged(AbilitySystemComponent->GetNumericAttribute(UHealthAttributeSet::GetHealthRegenerationAttribute()), 0.0f);  
    }    else  
    {  
       return false;  
    }}  
  
// Bind Stamina attribute delegates if the Ability System Component has the required Attribute Set -and- we are listening for Stamina attributes.  
if (ListenForStaminaAttributeSetChanges)  
{  
    if (AbilitySystemComponent->HasAttributeSetForAttribute(UStaminaAttributeSet::GetMaximumStaminaAttribute()))  
    {       MaximumStaminaChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UStaminaAttributeSet::GetMaximumStaminaAttribute()).AddUObject(this, &UAbilitySystemWidget::MaximumStaminaChanged);  
       CurrentStaminaChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UStaminaAttributeSet::GetCurrentStaminaAttribute()).AddUObject(this, &UAbilitySystemWidget::CurrentStaminaChanged);  
       StaminaRegenerationChangeDelegate = AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UStaminaAttributeSet::GetStaminaRegenerationAttribute()).AddUObject(this, &UAbilitySystemWidget::StaminaRegenerationChanged);  
  
       const float MaxStamina = AbilitySystemComponent->GetNumericAttribute(UStaminaAttributeSet::GetMaximumStaminaAttribute());  
       const float CurrentStamina = AbilitySystemComponent->GetNumericAttribute(UStaminaAttributeSet::GetCurrentStaminaAttribute());  
    // Call the Blueprint Events to initialize the values.  
       On_MaximumStaminaChanged(MaxStamina, 0.0f, CurrentStamina / MaxStamina);  
       On_CurrentStaminaChanged(CurrentStamina, 0.0f, CurrentStamina / MaxStamina);  
       On_StaminaRegenerationChanged(AbilitySystemComponent->GetNumericAttribute(UStaminaAttributeSet::GetStaminaRegenerationAttribute()), 0.0f);  
    }    else  
    {  
       return false;  
    }}  
  
return true;
```

MaximumHealthChanged()
```
const float CurrentHealth = AbilitySystemComponent->GetNumericAttribute(UHealthAttributeSet::GetCurrentHealthAttribute());  
  
On_MaximumHealthChanged(Data.NewValue, Data.OldValue, CurrentHealth / Data.NewValue);
```

CurrentHealthChanged()
```
const float MaxHealth = AbilitySystemComponent->GetNumericAttribute(UHealthAttributeSet::GetMaximumHealthAttribute());  
  
On_CurrentHealthChanged(Data.NewValue, Data.OldValue, Data.NewValue / MaxHealth);
```

HealthRegenerationChanged()
```
On_HealthRegenerationChanged(Data.NewValue, Data.OldValue);
```

MaximumStaminaChanged()
```
const float CurrentStamina = AbilitySystemComponent->GetNumericAttribute(UStaminaAttributeSet::GetCurrentStaminaAttribute());  
  
On_MaximumStaminaChanged(Data.NewValue, Data.OldValue, CurrentStamina / Data.NewValue);
```

CurrentStaminaChanged()
```
const float MaxStamina = AbilitySystemComponent->GetNumericAttribute(UStaminaAttributeSet::GetMaximumStaminaAttribute());  
  
On_CurrentStaminaChanged(Data.NewValue, Data.OldValue, Data.NewValue / MaxStamina);
```

StaminaRegenerationChanged()
```
On_StaminaRegenerationChanged(Data.NewValue, Data.OldValue);
```

ResetDelegateHandle()
```
if (IsValid(OldAbilitySystemComponent))  
{  
    OldAbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(Attribute).Remove(DelegateHandle);  
    DelegateHandle.Reset();  
}
```


# Character

