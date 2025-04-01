---
title: GAS GameplayEffect
date: 
tags:
  - 
categories: []
description: 
---

## GAS GameplayEffect

### 初始化

1. 我们在`AuraCharacter`里初始化了角色信息， `InitAbilityActorInfo`里有个`InitializeDefaultAttributes()`操作，这个操作在`AuraCharacterBase`里定义，具体代码为:

    ```cpp
    // 这个函数是对角色设置了Primary、Secondary、Vital属性的默认值
    void AAuraCharacterBase::InitializeDefaultAttributes() const
    {
    ApplyEffectToSelf(DefaultPrimaryAttributes, 1.f);
    ApplyEffectToSelf(DefaultSecondaryAttributes, 1.f);
    ApplyEffectToSelf(DefaultVitalAttributes, 1.f);
    }
    ```

2. 敌人属性初始化在`AuraEnemy`里的`InitAbilityActorInfo`，同样是调用父类的`InitializeDefaultAttributes()`，具体代码为:

    ```cpp
    void AAuraEnemy::InitAbilityActorInfo()
    {
     AbilitySystemComponent->InitAbilityActorInfo(this, this);
     Cast<UAuraAbilitySystemComponent>(AbilitySystemComponent)->AbilityActorInfoSet();

     InitializeDefaultAttributes();
    }
    ```

    然后我们就在继承自`AuraEnemy`的蓝图里配置要初始化的属性值，这时候，继承自这个蓝图的敌人就会有这些默认属性值了。
    我们这个例子是生成火球，所以我们再去火球的蓝图里配置GE。
    在GA蓝图里生成火球后，我们就能在火球重叠时触发对应的效果了。`AuraProjectile`里`OnSphereOverlap`函数里有个对`OtherActor`的
    `ApplyGameplayEffectSpecToSelf`函数，这个函数就是应用GE的函数。然后生成的`AuraprojectileSpell`的里面我们有配置`DamageEffectSpecHandle`，GE属性又在`AuraAttributeSet`里触发`PostGameplayEffectExecute`。
