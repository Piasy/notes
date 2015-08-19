#Rx (Reactive eXtention)

##细节
+  just, from等操作均是在创建时执行，而非subscribe时；create, defer等操作均是在subscribe时执行；create多次subscribe只会执行一次，defer多次subscribe会执行多次；
+  hot v.s. cold Observable
  +  cold：当Observable被subscribe时才开始发射item；（后来的subscriber同样会收到其subscribe之前发射的item；retrofit实现的是cold；）
  +  hot：创建之后就会开始发射item，不管是否被subscribe；（后来的subscriber不会收到其subscribe之前发射的item；）
+  map v.s. flatMap
+  将cold observable通过cache转换成hot之后，再在别处subscribe他们，有其应用场景：第一次subscribe时并不关心结果，但是后面某处想要获取结果，又不希望再次执行创建observable的过程；类似于预取思想；
+  怎么感觉有问题。。。
+  concat v.s. merge：concat不会让参数observable发射的item之间重叠，而merge可能会；concat传入的参数顺序是有影响的，merge没影响；

##Code review
+  [part I](http://artemzin.com/blog/rxjava-code-review-part-1)