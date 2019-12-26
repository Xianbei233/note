# Ali全家桶的项目操作笔记

## dva model in umi

umi中，model分全局和页面两种，在项目运行时，全局model将全量全时载入到唯一维护的Store中，而页面model将会按时按需载入到唯一的store中，因此页面model在其他页面可能因为载入顺序的原因而被能使用，因此model的namespace不能相同且不能跨页面使用，否则会发生载入顺序不同导致的数据不一致的问题。在开发者模式中，两者都是全量全时的载入。

## dva-loading

dva的插件，用于页面加载时的自动loading。loading对象会存在于全局Store中随着数据流进入组件，通过connect连接。