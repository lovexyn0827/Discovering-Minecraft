# Mixin疑难杂症解决记录

在编写CoreMod的过程中，调试Mixin无疑是最烦人的一个部分（之一），这里我将以解决我在实际Mod开发过程中遇到的问题为背景解释一些在Mixin开发过程中的一些疑难杂症，供大家参考。

## 2022.7.15：Redirect注入过程中出现数组访问越界

这一次的问题很令人摸不到头脑，而解决后又像是水贴一般。

崩溃堆栈如下

````
......（略略略）
Caused by: org.spongepowered.asm.mixin.transformer.throwables.InvalidMixinException: Unexpecteded ArrayIndexOutOfBoundsException whilst transforming the mixin class: [INJECT Applicator Phase -> messmod.mixins.json:ServerPlayNetworkHandlerMixin -> Apply Injections -> Inject -> messmod.mixins.json:ServerPlayNetworkHandlerMixin->@Redirect::originalIsHost()Z]
    at org.spongepowered.asm.mixin.transformer.MixinTargetContext.applyInjections(MixinTargetContext.java:1300)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.applyInjections(MixinApplicatorStandard.java:1042)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.applyMixin(MixinApplicatorStandard.java:395)
    at org.spongepowered.asm.mixin.transformer.MixinApplicatorStandard.apply(MixinApplicatorStandard.java:320)
    at org.spongepowered.asm.mixin.transformer.TargetClassContext.applyMixins(TargetClassContext.java:345)
    at org.spongepowered.asm.mixin.transformer.MixinProcessor.applyMixins(MixinProcessor.java:569)
    at org.spongepowered.asm.mixin.transformer.MixinProcessor.applyMixins(MixinProcessor.java:351)
    ... 27 more
Caused by: java.lang.ArrayIndexOutOfBoundsException: Index 0 out of bounds for length 0
    at org.spongepowered.asm.mixin.injection.code.Injector.checkCoerce(Injector.java:576)
    at org.spongepowered.asm.mixin.injection.code.Injector.validateParams(Injector.java:531)
    at org.spongepowered.asm.mixin.injection.invoke.RedirectInjector.injectAtInvoke(RedirectInjector.java:381)
    at org.spongepowered.asm.mixin.injection.invoke.RedirectInjector.inject(RedirectInjector.java:322)
    at org.spongepowered.asm.mixin.injection.code.Injector.inject(Injector.java:264)
    at org.spongepowered.asm.mixin.injection.struct.InjectionInfo.inject(InjectionInfo.java:385)
    at org.spongepowered.asm.mixin.transformer.MixinTargetContext.applyInjections(MixinTargetContext.java:1284)
    ... 33 more

````

这一看貌似是Mixin本身的问题，但真正的问题出在我们的代码中，Mixin至多只算是在异常处理上有所欠缺。出问题的代码如下（注入类为ServerPlayNetworkHandler）：

```java
@Redirect(method = {"onDisconnected", "onUpdateDifficulty", "onUpdateDifficultyLock"},	
		at = @At(value = "INVOKE", 
				target = "Lnet/minecraft/server/network/ServerPlayNetworkHandler"
                     	+ ";isHost()Z"
		)
)
private boolean originalIsHost() {
	return this.server.isHost(this.player.getGameProfile());
}
```

如果对Mixin不是很熟悉，你或许会觉得上面一段代码不应有任何问题，然而实际上Callback的方法定义应该是这样的：

````java
private boolean originalIsHost(ServerPlayNetworkHandler serverPlayNetworkHandler)
````

也就是说Callback中还需要接收一个this实例作为参数，细节！