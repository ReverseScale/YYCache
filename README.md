### Tips：

#### 特色：
去掉了异步访问的接口，尽量优化了同步访问的性能，用 OSSpinLock 来保证线程安全。
缓存内部用双向链表和 NSDictionary 实现了 LRU 淘汰算法。

#### 实现流程：
1）存值
Save 内存缓存 - Save 磁盘缓存
2）取值/查找
Check 内存存在否 - Check 磁盘存在否 - Save 入内存缓存 => nil
3）移除
Remove 内存缓存 - Remove 磁盘缓存

#### 内存缓存管理策略：
1）双向链表：数据结构缓存保存
2）缓存淘汰算法：使用LRU(least-recently-used) 算法来淘汰（清理）使用频率较低的缓存。
3）缓存清理策略：使用三个维度来标记，分别是count（缓存数量），cost（开销），age（距上一次的访问时间）。YYMemoryCache提供了分别针对这三个维度的清理缓存的接口。用户可以根据不同的需求（策略）来清理在某一维度超标的缓存。
4）互斥锁：保证线程安全

#### 磁盘缓存管理策略：
1）根据缓存数据的大小来采取不同的形式（>=20kb）的缓存：
🀇 数据库sqlite: 针对小容量缓存，缓存的data和元数据都保存在数据库里。
🀈 文件+数据库的形式: 针对大容量缓存，缓存的data写在文件系统里，其元数据保存在数据库里。
2）信号量：保证数据安全

#### 通用优化：
1）使用更底层且快速的 CFDictionary 而没有用 NSDictionary。
2）使用__has_include来检查Frameworks是否引入某个类。

==============

YYCache
==============

[![License MIT](https://img.shields.io/badge/license-MIT-green.svg?style=flat)](https://raw.githubusercontent.com/ibireme/YYCache/master/LICENSE)&nbsp;
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)&nbsp;
[![CocoaPods](http://img.shields.io/cocoapods/v/YYCache.svg?style=flat)](http://cocoapods.org/pods/YYCache)&nbsp;
[![CocoaPods](http://img.shields.io/cocoapods/p/YYCache.svg?style=flat)](http://cocoadocs.org/docsets/YYCache)&nbsp;
[![Support](https://img.shields.io/badge/support-iOS%206%2B%20-blue.svg?style=flat)](https://www.apple.com/nl/ios/)&nbsp;
[![Build Status](https://travis-ci.org/ibireme/YYCache.svg?branch=master)](https://travis-ci.org/ibireme/YYCache)

High performance cache framework for iOS.<br/>
(It's a component of [YYKit](https://github.com/ibireme/YYKit))

Performance
==============

![Memory cache benchmark result](https://raw.github.com/ibireme/YYCache/master/Benchmark/Result_memory.png
)

![Disk benchmark result](https://raw.github.com/ibireme/YYCache/master/Benchmark/Result_disk.png
)

You may [download](http://www.sqlite.org/download.html) and compile the latest version of sqlite and ignore the libsqlite3.dylib in iOS system to get higher performance.

See `Benchmark/CacheBenchmark.xcodeproj` for more benchmark case.


Features
==============
- **LRU**: Objects can be evicted with least-recently-used algorithm.
- **Limitation**: Cache limitation can be controlled with count, cost, age and free space.
- **Compatibility**: The API is similar to `NSCache`, all methods are thread-safe.
- **Memory Cache**
  - **Release Control**: Objects can be released synchronously/asynchronously on main thread or background thread.
  - **Automatically Clear**: It can be configured to automatically evict objects when receive memory warning or app enter background.
- **Disk Cache**
  - **Customization**: It supports custom archive and unarchive method to store object which does not adopt NSCoding.
  - **Storage Type Control**: It can automatically decide the storage type (sqlite / file) for each object to get
      better performance.


Installation
==============

### CocoaPods

1. Add `pod 'YYCache'` to your Podfile.
2. Run `pod install` or `pod update`.
3. Import \<YYCache/YYCache.h\>.


### Carthage

1. Add `github "ibireme/YYCache"` to your Cartfile.
2. Run `carthage update --platform ios` and add the framework to your project.
3. Import \<YYCache/YYCache.h\>.


### Manually

1. Download all the files in the YYCache subdirectory.
2. Add the source files to your Xcode project.
3. Link with required frameworks:
	* UIKit
	* CoreFoundation
	* QuartzCore
	* sqlite3
4. Import `YYCache.h`.


Documentation
==============
Full API documentation is available on [CocoaDocs](http://cocoadocs.org/docsets/YYCache/).<br/>
You can also install documentation locally using [appledoc](https://github.com/tomaz/appledoc).


Requirements
==============
This library requires `iOS 6.0+` and `Xcode 8.0+`.


License
==============
YYCache is provided under the MIT license. See LICENSE file for details.


<br/><br/>
---
中文介绍
==============
高性能 iOS 缓存框架。<br/>
(该项目是 [YYKit](https://github.com/ibireme/YYKit) 组件之一)

性能
==============

iPhone 6 上，内存缓存每秒响应次数 (越高越好):
![Memory cache benchmark result](https://raw.github.com/ibireme/YYCache/master/Benchmark/Result_memory.png
)

iPhone 6 上，磁盘缓存每秒响应次数 (越高越好):
![Disk benchmark result](https://raw.github.com/ibireme/YYCache/master/Benchmark/Result_disk.png
)

推荐到 SQLite 官网[下载](http://www.sqlite.org/download.html)和编译最新的 SQLite，替换 iOS 自带的 libsqlite3.dylib，以获得更好的性能。

更多测试代码和用例见 `Benchmark/CacheBenchmark.xcodeproj`。


特性
==============
- **LRU**: 缓存支持 LRU (least-recently-used) 淘汰算法。
- **缓存控制**: 支持多种缓存控制方法：总数量、总大小、存活时间、空闲空间。
- **兼容性**: API 基本和 `NSCache` 保持一致, 所有方法都是线程安全的。
- **内存缓存**
  - **对象释放控制**: 对象的释放(release) 可以配置为同步或异步进行，可以配置在主线程或后台线程进行。
  - **自动清空**: 当收到内存警告或 App 进入后台时，缓存可以配置为自动清空。
- **磁盘缓存**
  - **可定制性**: 磁盘缓存支持自定义的归档解档方法，以支持那些没有实现 NSCoding 协议的对象。
  - **存储类型控制**: 磁盘缓存支持对每个对象的存储类型 (SQLite/文件) 进行自动或手动控制，以获得更高的存取性能。


安装
==============

### CocoaPods

1. 在 Podfile 中添加 `pod 'YYCache'`。
2. 执行 `pod install` 或 `pod update`。
3. 导入 \<YYCache/YYCache.h\>。


### Carthage

1. 在 Cartfile 中添加 `github "ibireme/YYCache"`。
2. 执行 `carthage update --platform ios` 并将生成的 framework 添加到你的工程。
3. 导入 \<YYCache/YYCache.h\>。


### 手动安装

1. 下载 YYCache 文件夹内的所有内容。
2. 将 YYCache 内的源文件添加(拖放)到你的工程。
3. 链接以下的 frameworks:
	* UIKit
	* CoreFoundation
	* QuartzCore
	* sqlite3
4. 导入 `YYCache.h`。


文档
==============
你可以在 [CocoaDocs](http://cocoadocs.org/docsets/YYCache/) 查看在线 API 文档，也可以用 [appledoc](https://github.com/tomaz/appledoc) 本地生成文档。


系统要求
==============
该项目最低支持 `iOS 6.0` 和 `Xcode 8.0`。


许可证
==============
YYCache 使用 MIT 许可证，详情见 LICENSE 文件。


相关链接
==============
[YYCache 设计思路与技术细节](https://blog.ibireme.com/2015/10/26/yycache/)


