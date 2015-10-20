# 《Test Driven Development: By Example》一书

## 核心思想
+  Rules
  +  Write new code only if you first have a failing automated test.
  +  Eliminate duplication.
+  Red/green/refactor. The TDDs mantra.
  +  Red: write a little test that doesn’t work, perhaps doesn’t even compile at first
  +  Green: make the test work quickly, committing whatever sins necessary in the process
  +  Refactor: eliminate all the duplication created in just getting the test to work

## By Example
+  首先，来了一个需求，不是思考需要一个什么对象，需要一个什么方法，而是需要一个什么测试，来验证我们的实现是正确的
+  更进一步的，是需求可测试化：需求不是一句话，而是一个todo list，其实就是所有的（尽量）测例，有什么需求都通过测例来描述，在什么情况下软件应该是什么行为，要细化到可以直接翻译为代码
+  思考需要解决的问题的时候，都是以：我需要什么样的测试以验证实现正确开始，而不是如何实现
+  测试化的需求可能繁简程度不一，那么就需要先从简单的开始，同时学会把复杂的测例拆分为简单的测例
+  为尚不存在的代码编写测例时，首先假设它有完美的接口满足其功能，同时先不要管封装、数据类型等复杂问题，让测试代码尽量简单，例如：
	```java
	public void testMultiplication() { 
		Dollar five= new Dollar(5); 
		five.times(2);
		assertEquals(10, five.amount);
	}
	```
+  一步一步来，先让让测例写出来，但同时要记下需要改善的地方，先写出一个可以跑，但是失败的测例，然后再逐步实现、逐步改善测例与实现
+  编写的测例首先应该是不能编译的，借助IDE，我们可以很方便的添加相应的类、成员、方法，使得测例能够编译，注意！这一步不要加入任何实现，仅仅让测例可编译，一旦加入了实现，就容易沉浸在实现中了
+  测例运行起来当然是失败的，但这是巨大的进步！从“实现XX功能”，到“让这个测例通过”，简单了无数倍！
+  TDD五部曲，有点naive，但值得借鉴
  +  Add a little test
  +  Run all tests and fail
  +  Make a little change，最小改动，例如直接把amount初始值赋值为10
  +  Run the tests and succeed
  +  Refactor to remove duplication


+  好代码的“天敌”：依赖与重复
  +  依赖：上面的测例显然依赖于实现，改实现就必须要改测例，而这并不是我们想要的，我们的目标是编写有意义（能验证实现正确性）的测例，但是变更实现细节不需要修改测例，目前看来有点难；实际开发也是如此，如果代码依赖于某一组件的实现细节，那以后想要替换另一组件，就很麻烦；
  +  重复：If dependency is the problem, duplication is the symptom. 代码的重复往往起源于重复的逻辑，例如逻辑一样的表达式出现在多个地方；而对象（面向对象）天生就是用来把重复的逻辑抽象出来的，别辜负了对象；
  +  还好，在程序中，消除重复，就能消除依赖
+  找到重复，消除重复
  +  和实际代码的重复不太一样，不是基本相同的逻辑在重复；测例与实现的重复在于数据的重复，例如直接`int amount= 10;`和测例的测试数据是重复的
  +  书中的例子非常简单，但道理很深刻：TDD不是关于一些小的、简单的步骤，而是关于如何能够把难题拆分为小的、简单的步骤！
  +  其实当能够把每一步拆分得不能再小时，就表明对问题的理解达到了一定的程度，此时当然知道应该把问题拆分为何种粒度的子问题，保持思考！
+  让测例能够通过的合适的Dollar类代码，谁都会写，但是这个思考的过程却未必，而这正是践行TDD应该注意的地方！
+  完成上步后，就是时候考虑实现的优雅性了，将其也加入到todo list中吧！


+  一步一步来！