## 前端的内存控制

#### 为什么要关注内存，关注性能
	前端开发，除了创建前端界面呈现给用户，也需要关注用户的使用体验。
	在一些老旧的设备上，如果出现内存泄漏，很容易导致用户的设备白屏崩溃等问题。
	同时，因为浏览器已经帮我们做了很多，很容易就忽略掉内存问题。

### 浏览器的内存回收
  - 方式：标记清除
		https://www.ituring.com.cn/book/tupubarticle/10955
	- 浏览器做了什么
		基础GC：
		1. 遍历所有可访问的对象。
		2. 回收已不可访问的对象。
		GC优化
		分代回收 (Generation GC)
		增量GC (incremental GC)

### 常见的泄漏场景
  1. DOM 泄漏
  2. 对象泄漏
	3. 闭包 (事件处理回调等)

### 编码习惯
  
1. 定时清理listener、event等发布订阅/事件监听;

	```ts
	// 现代浏览器已经会在移除dom时将绑定该dom的事件一并移除，但如果在document上，那么需要重新考虑移除listener
	// e.g.  In vue component created/mounted hook
	someEleOrDocument.addEventListener('click', someFunc);
	// ... In Vue component beforeDestroyed/destroyed hook
	someEleOrDocument.removeEventListener('click', someFunc);
	```

2. 定时移除全局时间监听

	```ts
	// Vue 2 中很常用的一个全局事件策略是 Vue.prototype.$EventBus = new Vue();
	// 这么一来可以通过 $emit(evt: string, param: any), $on(evt: string, callback: Funcion) 进行一个全局的发布订阅

	// e.g. In vue component created/mounted hook
	this.$EventBus.$on('some_event_name', this.someFunction);
	// ... In Vue component beforeDestroyed/destroyed hook
	this.$EventBus.$off('some_event_name', this.someFunction);
	// or simply remove all subscribed callback
	// this.$EventBus.$off('some_event_name');
	```

3. 观察者使用需要及时移除

	```ts
	// Observer cases
	function startObserve() {
		observer = new MutationObserver(someCallBackFuncion);
		observer.observe(someEl);
	}
	// ... after finish it's task
	observer.unobserve(someEl);
	// ... or
	observer.disconnect();
	```

### 当前项目中用到的一些节省内存的模式(性能/存储取舍)

1. #### 数据缓存, 控制js数据量

   有些场景下，有些短时间内不会变化的数据会采用缓存的策略。可以节省一些时间上的开销（网络请求/读取本地数据库等）；但是如果缓存的数据变更频繁，也会成为比较占据内存的点；在一个频繁读取用户信息场景，我这里采取了一个简单的LRU的设计去缓存信息：
	
    ```ts
    /**
     * @description simple lru cache sample
    */

    export type CacheKey = String | Symbol | Number;

    export interface CacheItem {
      key: CacheKey;
    }

    export default class LRUCache {
      private cacheList: CacheItem[];
      private cacheSize: number;

      constructor(size: number) {
        this.cacheList = [];
        this.cacheSize = size;
      }

     /**
      * @description get cache. if cache exist, prioritize to head of list
      * @param key CacheItem key
      * @returns CacheItem | Null
      */
      public getCache(key: CacheKey): null | CacheItem {
        let targetKey;
        for (let i = 0; i < this.cacheList.length; i++) {
          const cacheItem = this.cacheList[i];
          if (cacheItem.key === key) {
            targetKey = i;
            break;
          }
        }
        if (!targetKey) {
          return null;
        }
        const [target] = this.cacheList.splice(targetKey, 1);
        this.cacheList.push(target);
        return target;
      }

      /**
      * @description set cache to list.
      * @param item { CacheItem }
      * @returns void
      */
      public setCache(item: CacheItem): void {
        const isCacheExist = this.getCache(item.key);
        if (isCacheExist !== null) {
          return;
        }
        if (this.cacheList.length >= this.cacheSize) {
          this.cacheList.pop;
        }
        this.cacheList.unshift(item);
      }

      /**
      * @description update cache list size.
      * @param size { number }
      */
      public setCacheSize(size: number) {
        if (size <= 0) {
          throw RangeError('cache size should bigger than 0');
        }
        if (size >= this.cacheSize) {
          this.cacheSize = size;
        } else {
          this.cacheList.length = size;
        }
      }

      public destroy() {
        this.cacheList = [];
        this.cacheSize = 0;
      }
    }
    ```

  原本的方式是将所有的数值按照key	- value的方式，存储到了一个对象/map中。虽然这样的读取很快，但是在时间久了之后会存储几千上万条用户信息，占了比较大的空间。采用这种缓存队列，可以一定程度上限制缓存的数据量，缺点则是对接口的请求会更加频繁，读取上也会更慢(如果队列设置过长，那么优先级越低的数据要遍历越久).

2. #### 虚拟列表，控制dom数目

    一个简单的列表条目定高demo:

    ```html
    <div style="height: 100%; overflow: hidden;">
      <span class="sticky-hint"> current list dom count: {{ count }} </span>
      <!-- virtual list container -->
      <div class="list-container" ref="listcontainer" @scroll="computeListRange">
        <!-- list phantom, fill the size -->
        <div class="list-phantom" :style="{ height: listHeight }"></div>
        <!-- list content, visble list content -->
        <ul class="list-content" ref="listcontent" :style="visibleArea">
          <li v-for="item in visbleList" :key="item">{{ item }}</li>
          <div ref="bottom">bottom</div>
        </ul>
      </div>
    </div>
    ```

    ```js
    ```