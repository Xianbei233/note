# Ali全家桶的项目操作笔记

## dva model in umi

umi中，model分全局和页面两种，在项目运行时，全局model将全量全时载入到唯一维护的Store中，而页面model将会按时按需载入和卸载到唯一的store中，因此页面model在其他页面无法使用。在开发者模式中，两者都是全量全时的载入。

