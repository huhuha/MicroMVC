### 关于module

### 相关文件
```
|- Framework	框架
	|- Libraries	公共类库
		|- Curl.php 	http请求类
	|- Modles	框架逻辑
		|- LocalCurl.php 	本地Curl模仿类
|- ModuleA
|- ModuleB
```

### 使用方法
1. 框架层做了 Module 之间的资源隔离，在运行 ModuleA 时，MoudleA内的代码是无法加载 ModuleB 内的代码的（框架层面禁止加载，如果非要破坏这种控制，就需要自己写类加载器）。这样做是为了方便未来可能的 Module 迁移。此时如果想迁移，只用把 Framework、public和想迁移的Module目录即可。
1. 但是并不是所有的 Module 都会有迁移需求，为了尽可能减少性能损耗和资源消耗，框架提供了一个 LocalCurl 类。该类提供的方法名、使用方式都与公共库内的 Curl 类相同。所以可以使用 LocalCurl 类来进行本地的 Module 之间的数据交互。使用样例如下：
```
$lc = new LocalCurl();
$lc->setAction('test', 'http://127.0.0.1/xhprof/xhprof/run');
var_dump($lc->get('test')->body());
```
1. 此时如果想迁移 Module ，也能非常方便的进行。只需要在项目代码里全文匹配字符串"LocalCurl",然后替换成"Curl"，在修改访问域名地址即可完成代码层面的迁移。然后再按上述方式物理迁移 Module 即可。

### 为什么这么做？
 在实际项目中，尤其是业务数量多、迭代快的项目里，这种需求尤其频繁。    
 一个新需求到来，刚开始可能只是一个小工具，此时一般并不会新申请一个新的域名或新的一套资源。但是随着时间的推移，这些小需求是有概率长成一个大项目的，此时想从原来项目中拆出来会是一个非常头痛的事情,这么做就是为了避免这样的“头痛”发生，即使只用到一次也会非常的值得。     
 那有些人可能会说，任何一个独立的新需求我是不是能快速申请一个三级域名就新建项目，不就解决了潜在的迁移风险吗？这在小团队中看起来还是比较可行的，但是当公司规模大起来，这一系列流程非常繁琐，时间周期也会比较久,完全跟不上web迭代的速度要求。最重要的是，新需求成长成大的独立项目真的不是一个很大概率的事件。    
 而实际经常出现的事情是： A 业务做了一个工具，本来只是想给 A 业务用，后来 B 业务发现这个工具很好用，也想用，就需要扩展该工具甚至把该工具独立出来。 而并不是业务 A 一开始做这个工具的时候就设想了其他业务都会想用这个工具，而把该工具做的通用性很强、扩展性很强，这样臆造需求也是经常出现的问题，很大程度上都会存在过度设计和浪费研发资源的情况。  
 现在这个框架在几个项目中都运行良好，尤其是在大后台项目中，很多工具都可以先放在一个环境中开发和使用，真到这个工具能长大的时候，再迁移也完全不是困难的事情。