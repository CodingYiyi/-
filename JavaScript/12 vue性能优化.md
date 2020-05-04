# vue 性能优化tips

## watch 和 computed 的应用场景

```
// watch 写法
watch:{
	// watchedData属性需要提前在 data 中声明，即提前绑定到vue实例上，否则会报错。
	// watch 的属性接受两个参数：新值，旧值
	watchedData:function(newVal,oldVal){
		// do someting 
	}
}
```

```
// computed 写法
computed:{
	// 计算属性不需要提前在 data 中声明
	// 计算属性总是需要 return 一个值，否则请考虑是否正确地使用了计算属性，是否一个简单的method就可以了
	computedData(){
		return ……
	}
}
```

**watch**：更多的是「观察」的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行指定的操作；

**computed**：是计算属性，**依赖其它属性值**，并且 computed 的值有缓存，只有它依赖的属性值发生改变时才会重新计算 computed  的值，否则直接从缓存中返回上次的值；

watch 和 computed 都有监听某个值的变化，然后做出相应操作的功能，而且计算属性还有缓存的功能，性能更佳。

那是否计算属性可以完全替代watch呢？

官网提供的例子就是个很好的demo：[什么时候使用watch？](https://cn.vuejs.org/v2/guide/computed.html#侦听器)。总结下来就是watch可以提供更细力度的控制，应对更复杂的场景（异步、函数节流等），而计算属性由于每次都需要同步返回一个值，所以上述场景就无法应对了。


## 2.你真的需要把所有的数据都定义在data中吗？
由于template中的动态数据都需要从vue实例上取，所以我们平时开发的时候不假思索就将所有页面用的的数据放置到data中：

```
data(){
	return {
		data1,
		data2,
		data3,
		……
		dataN
	}
}
```
但是，这样做真的合适吗？或者说所有页面上要展示的数据，都要通过这种方式吗？

vue在依赖收集阶段会遍历data返回的对象的属性，增加watcher，特别是如果属性值为复杂对象，则会递归遍历，这是很消耗性能的一件事，我们应尽可能只收集真正需要的动态数据。那什么样的数据不需要添加依赖呢？举个🌰：

一个页面需要省市区的数据，如果这个数据是静态文件返回的，如下图：

```
import { provinces } from '@/util/consts';

export default {
  data() {
    return {
      myData: provinces,
    };
  },
}
```

上例中，provinces 的值是不会发生变化的，所以vue依赖收集过程应该剔除它（收集了也没有用啊），如下图：

```
import { provinces } from '@/util/consts';

export default {
  data() {
  	this.myData = provinces
   	return {
      ……
    };
  },
}
```
简单地绑定到this上即可。
可能有人会问，那如果省市区数据时是通过接口异步返回的呢？依旧可以！如下图：

```
export default {
	data:()=>({myData: {}}),
	async created() {
		provinces = await axios.get("/api/users");
		this.myData = Object.freeze(provinces);  }
}
```
这样写，vue就不会通过 Object.defineProperty 对数据 provinces 进行劫持。

**注意：此时直接修改this.myData不会触发更新，但是重新赋值还是会触发视图更新的，因为我们只是冻结了 provinces 的值。**

## 3. v-for key的正确使用







