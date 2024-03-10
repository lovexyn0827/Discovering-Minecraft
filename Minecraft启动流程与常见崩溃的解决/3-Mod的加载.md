---
keywords: Minecraft, Mod, 流程, 加载, lovexyn0827
desc: Minecraft的Mod的加载流程
id: fix_minecraft_crash:3
draft: false
---

# Mod的加载

### Fabric Loader

#### 加载Mod信息并验证兼容性

```
[19:05:51] [main/INFO]: Loading Minecraft 1.18.2 with Fabric Loader 0.14.8
[19:05:52] [main/INFO]: Loading 8 mods:
    - carpet 1.4.66
    - fabricloader 0.14.8
    - java 17
    - litematica 0.11.3
    - malilib 0.12.1
    - minecraft 1.18.2
    - minihud 0.22.0
    - tweakeroo 0.13.3
[19:05:52] [main/INFO]: SpongePowered MIXIN Subsystem Version=0.8.5 Source=file:/K:/minecraft/libraries/net/fabricmc/sponge-mixin/0.11.4+mixin.0.8.5/sponge-mixin-0.11.4+mixin.0.8.5.jar Service=Knot/Fabric Env=CLIENT
[19:05:52] [main/INFO]: Compatibility level set to JAVA_16
[19:05:52] [main/INFO]: Compatibility level set to JAVA_17
```

这时Mod加载器会在mods文件夹中查找Mod文件并读取其元数据（说白了，就是版本，兼容性等信息）。紧接着，加载器会分析这些信息并检查加载的Mod与Minecraft的版本是否兼容、需要的依赖是否存在以及是否存在相互冲突的Mod。

如果Minecraft版本不符、依赖缺失或发现了会造成致命冲突的Mod，游戏会在此处崩溃并给出原因。

![1672247632925](media/1672247632925.png)

如果发现了可能会相互冲突的Mod或推荐安装的Mod不存在，一条警告会被发出，这时继续运行游戏可能会导致问题。

```
[00:19:04] [main/WARN]: Warnings were found! 
 - Conflicting versions found for fabric-lifecycle-events-v1: used 1.2.0+74cc3b203a, also found 1.2.0+74cc3b2095
 - Conflicting versions found for fabric-networking-v0: used 0.1.10+e00ecb5f95, also found 0.1.10+e00ecb5f3a
 - Mod 'MessMod' (messmod) conflicts with any version of mod 'Lithium' (lithium), but a matching version is present: 0.6.0!
     - While this won't prevent you from starting the game, the developer(s) of 'MessMod' (messmod) have found that version 0.6.0 of 'Lithium' (lithium) conflicts with their mod.
     - It is heavily recommended to remove one of the mods.
     - Mod 'Lithium' (lithium) v0.6.0 is being loaded from the user's mod directory.
 - Conflicting versions found for fabric-resource-loader-v0: used 0.3.1+facf3bbf3a, also found 0.3.1+facf3bbf95
 - Mod 'Giselbaers Durability Viewer' (durabilityviewer) recommends version 1.1.7 or later of mod modupdater, which is missing!
     - You must install version 1.1.7 or later of modupdater.
```

需要注意的是，这些兼容性信息是Mod作者定义帮助用户确定问题所在的，而这些定义并不一定完全正确。有时Mod会兼容比作者的规定更多的Minecraft版本，而作者认为可能发生的冲突在Mod版本更新后也不一定会真正发生。因此，我们当然也可以修改这些信息进行强行适配，尽管并不推荐这样做。反过来，有时作者也会遗漏掉一些潜在的冲突，这时游戏可能会在接下来的运行中崩溃。

#### 加载AccessWidener

除非误用了未混淆的Mod，此处几乎不会出错，因为这里只是测试AccessWidener格式是否正确及运行环境是否匹配，而格式错误在Mod测试时就会被发现。同时，这也意味着进一步的错误不会被及时发现，游戏仍有可能因Mod兼容等问题在稍后崩溃。

#### 解析Mixin

这时加载器会解析Mod中的Mixin（用于修改游戏本体或其他Mod内部代码），此阶段很少会出错。

#### 启动游戏

这时Mod准备过程已经结束，原版的游戏本体会被启动。

#### 应用Mixin

这时是兼容性验证后出错概率最大的环节。如果Mod的运行环境与测试环境不同，Mixin在被应用时可能遇到找不到类、方法或字段以及修改冲突（如两个Mod同时重定向一个方法调用）等问题。

Mixin的应用随着类的加载进行。类加载的时机与Minecraft版本，模组安装情况等有关，很难统一。这里给出了1.16.4版本中的情况：

1. 加载渲染必须类，方块，实体类型，各种方块实体等类。
2. 打开世界列表时，加载内置服务端，世界信息读取，指令定义等类。
3. 打开存档时，加载服务端逻辑相关类。
4. 世界载入后，类加载基本完成，但还有爆炸等少数类未加载。

如果发生了找不到类、方法或字段，局部变量表变更等错误，游戏会崩溃。

```
java.lang.RuntimeException: Mixin transformation of net.minecraft.world.explosion.Explosion failed
    at net.fabricmc.loader.impl.launch.knot.KnotClassDelegate.getPostMixinClassByteArray(KnotClassDelegate.java:418)
    at net.fabricmc.loader.impl.launch.knot.KnotClassDelegate.tryLoadClass(KnotClassDelegate.java:322)
    at net.fabricmc.loader.impl.launch.knot.KnotClassDelegate.loadClass(KnotClassDelegate.java:218)
    at net.fabricmc.loader.impl.launch.knot.KnotClassLoader.loadClass(KnotClassLoader.java:145)
    at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:520)
    at net.minecraft.server.world.ServerWorld.createExplosion(ServerWorld.java:926)
    at net.minecraft.world.World.createExplosion(World.java:514)
    at net.minecraft.entity.TntEntity.explode(TntEntity.java:89)
    at net.minecraft.entity.TntEntity.tick(TntEntity.java:77)
    at net.minecraft.server.world.ServerWorld.tickEntity(ServerWorld.java:619)
    at net.minecraft.world.World.tickEntity(World.java:487)
    at net.minecraft.server.world.ServerWorld.method_31420(ServerWorld.java:338)
    at net.minecraft.world.EntityList.forEach(EntityList.java:78)
    at net.minecraft.server.world.ServerWorld.tick(ServerWorld.java:315)
    at net.minecraft.server.MinecraftServer.tickWorlds(MinecraftServer.java:873)
    at net.minecraft.server.MinecraftServer.tick(MinecraftServer.java:828)
    at net.minecraft.server.integrated.IntegratedServer.tick(IntegratedServer.java:93)
    at net.minecraft.server.MinecraftServer.runServer(MinecraftServer.java:683)
    at net.minecraft.server.MinecraftServer.method_29739(MinecraftServer.java:293)
    at java.base/java.lang.Thread.run(Thread.java:833)
Caused by: org.spongepowered.asm.mixin.transformer.throwables.MixinTransformerError: An unexpected critical error was encountered
    at org.spongepowered.asm.mixin.transformer.MixinProcessor.applyMixins(MixinProcessor.java:392)
    at org.spongepowered.asm.mixin.transformer.MixinTransformer.transformClass(MixinTransformer.java:234)
    at org.spongepowered.asm.mixin.transformer.MixinTransformer.transformClassBytes(MixinTransformer.java:202)
    at net.fabricmc.loader.impl.launch.knot.KnotClassDelegate.getPostMixinClassByteArray(KnotClassDelegate.java:413)
    ... 19 more
Caused by: org.spongepowered.asm.mixin.injection.throwables.InjectionError: LVT in net/minecraft/world/explosion/Explosion::getExposure(Lnet/minecraft/util/math/Vec3d;Lnet/minecraft/entity/Entity;)F has incompatible changes at opcode 183 in callback messmod.mixins.json:ExplosionMixin from mod messmod->@Inject::renderLines(Lnet/minecraft/util/math/Vec3d;Lnet/minecraft/entity/Entity;Lorg/spongepowered/asm/mixin/injection/callback/CallbackInfoReturnable;Lnet/minecraft/util/math/Box;DDDDDIIFFFDDDLnet/minecraft/util/math/Vec3d;)V.
 Expected: [Lnet/minecraft/util/math/Box;, D, D, D, D, D, I, I, F, F, F, D, D, D, Lnet/minecraft/util/math/Vec3d;]
    Found: [Lnet/minecraft/util/math/Box;, D, D, D, D, D, I, I, D, D, D, D, D, D, Lnet/minecraft/util/math/Vec3d;]
Available: [Lnet/minecraft/util/math/Box;, D, D, D, D, D, I, I, D, D, D, D, D, D, Lnet/minecraft/util/math/Vec3d;]
    at org.spongepowered.asm.mixin.injection.callback.CallbackInjector.inject(CallbackInjector.java:547)
    at org.spongepowered.asm.mixin.injection.callback.CallbackInjector.inject(CallbackInjector.java:497)
    at org.spongepowered.asm.mixin.injection.code.Injector.inject(Injector.java:276)
    at org.spongepowered.asm.mixin.injection.struct.InjectionInfo.inject(InjectionInfo.java:445)
    at org.spongepowered.asm.mixin.transformer.MixinTargetContext.applyInjections(MixinTargetContext.java:1378)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.applyInjections(MixinApplicatorStandard.java:1052)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.applyMixin(MixinApplicatorStandard.java:400)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.apply(MixinApplicatorStandard.java:325)
    at org.spongepowered.asm.mixin.transformer.TargetClassContext.apply(TargetClassContext.java:421)
    at org.spongepowered.asm.mixin.transformer.TargetClassContext.applyMixins(TargetClassContext.java:403)
    at org.spongepowered.asm.mixin.transformer.MixinProcessor.applyMixins(MixinProcessor.java:363)
    ... 22 more

```

注意，这种崩溃属于Mod的一个Bug，建议将其报告给Mod作者。

当然也有不致命的错误，这时Mod加载器会发出一条警告，此时继续运行游戏和可能会导致一些问题，包括功能失效’游戏崩溃等。

```
[00:19:17] [main/WARN]: @ModifyConstant conflict. Skipping mixins.tweakeroo.json:MixinStructureBlockBlockEntity->@ModifyConstant::overrideMaxSize(I)I with priority 999, already redirected by carpet-tis-addition.mixins.json:rule.structureBlockLimit.StructureBlockBlockEntityMixin->@ModifyConstant::structureBlockLimitNegative(I)I with priority 1000
```

除非有进一步的错误发生，这种问题通常不必报告给Mod作者，因为此时作者很可能已经注意到兼容性问题并进行了一定的修改放置游戏崩溃。

对于一般用户，可以通过日志中与Mixin有关的报错来查找错误原因。很多时候，如果在报错中指找到了有关一个Mod的线索，那么崩溃可能是因为Mod与Minecraft本体版本不兼容；如果报错中存在两个Mod，那么崩溃很可能是两个Mod冲突的结果。

### Forge

说实话不大了解，不过印象中大体上与Fabric还是相似的。如有明显的不同之处的话会补上