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

## 2022.10.2：ModifyVariable无法修改方法参数中用到的变量

今天写的一个功能需要大概是要对MC本体进行如下修改：

原有

````java
AbstractMinecartEntity abstractMinecartEntity = AbstractMinecartEntity.create(world, 
		(double)blockPos.getX() + 0.5D, 
		(double)blockPos.getY() + 0.0625D + d, 
		(double)blockPos.getZ() + 0.5D, this.type);
````

修改后

````java
AbstractMinecartEntity abstractMinecartEntity = AbstractMinecartEntity.create(world, 
		(double)blockPos.getX() + 0.5D, 
		(double)blockPos.getY() + 0.0625D + localvar$zfp000$adjustHeight(d, context), 
		(double)blockPos.getZ() + 0.5D, this.type);
````

于是，我原以为像下面这样写就可以了

````java
@ModifyVariable(method = "useOnBlock",
			at = @At(value = "INVOKE",
					target = "Lnet/minecraft/entity/vehicle/AbstractMinecartEntity;"
							+ "create(Lnet/minecraft/world/World;DDDLnet/minecraft/entity/vehicle/AbstractMinecartEntity$Type;)"
							+ "Lnet/minecraft/entity/vehicle/AbstractMinecartEntity;", 
					shift = At.Shift.BEFORE
					),
			name = "d"
	)
````

但是，实际测试时却发现这样做似乎没有任何效果。开始，我在调用create()方法的那一行放置了一个断点，发现原始变量确实没有被修改。接着用JD反编译了一下Mixin的输出类，看起来注入方法似乎确实是已经被正确注入了。

```java
d = localvar$zfp000$adjustHeight(d, context);
AbstractMinecartEntity abstractMinecartEntity = AbstractMinecartEntity.create(world,
		blockPos.getX() + 0.5D, 
		blockPos.getY() + 0.0625D + d, 
		blockPos.getZ() + 0.5D, this.type);
```

后来我确认了注入方法确实是正确的，但这样问题就变得更加扑朔迷离。干耗了将近一个小时后，我抱着试试看的想法向create()方法后面又放了一个断点，结果是这时变量d已经被正确地修改了，这给了我一个新思路：

我们知道，JVM字节码中带参数方法调用大概会长这样（以`this.getStuff(2 + i1, obj1, obj2.arg)`为例）：

````assembly
00	aload_0		//this
01	iconst_2	//2
02	iload 4		//i1
04	iadd
05	aload 5		//obj1
07	aload 6		//obj2
09	getfield #114
12	invokevirtual #514
````

即

1. 加载this（对于实例方法）
2. 加载各参数（入栈），包括一些算术运算、字段获取与方法调用等预处理操作
3. 最终调用方法

如果我们修改参数i1，我们至少要在位置02之前，即JVM将i1的值存储为一个副本作为方法参数之前进行操作，否则变量的修改将不能再应用于实际的方法参数（~~除非把原来的pop出来再将新值入栈实现更新~~）。但是，Mixin中INVOKE在这里匹配的是invokevirtual指令之前，即11-12位置之间，这时再进行变量修改为时已晚。

回到原问题，有了这么一个思路，就用javap反汇编一下输出的类文件，果然不出所料：

````asm
     108: aload_2
     109: aload_3
     110: invokevirtual #120                // Method net/minecraft/util/math/BlockPos.getX:()I
     113: i2d
     114: ldc2_w        #113                // double 0.5d
     117: dadd
     118: aload_3
     119: invokevirtual #123                // Method net/minecraft/util/math/BlockPos.getY:()I
     122: i2d
     123: ldc2_w        #124                // double 0.0625d
     126: dadd
     127: dload         7
     129: dadd
     130: aload_3
     131: invokevirtual #128                // Method net/minecraft/util/math/BlockPos.getZ:()I
     134: i2d
     135: ldc2_w        #113                // double 0.5d
     138: dadd
     139: aload_0
     140: aload_0
     141: dload         7
     143: aload_1
     144: invokespecial #132                // Method localvar$zfp000$adjustHeight:(DLnet/minecraft/item/ItemUsageContext;)D
     147: dstore        7
     149: getfield      #32                 // Field type:Lnet/minecraft/entity/vehicle/AbstractMinecartEntity$Type;
     152: invokestatic  #136                // Method net/minecraft/entity/vehicle/AbstractMinecartEntity.create:(Lnet/minecraft/world/World;DDDLnet/minecraft/entity/vehicle/AbstractMinecartEntity$Type;)Lnet/minecraft/entity/vehicle/AbstractMinecartEntity;
````

其实解决起来也很容易，另找合适的注入点即可。但是，这一问题确实是一个大坑，还请大家注意。