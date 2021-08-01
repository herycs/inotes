# Spring QA

> 生命周期(启动流程)｜Autocongifition 实现｜ 三级缓存(解决三个问题)｜ AOP实现阶段｜ Bean类型 | 如何插入拦截器 ｜ 设定切面

## 三级缓存

singletonObjects -> beanName <> instaance

earlySingletonObjects -> 初次实例化完 new instance

singletonFactories -> 三级缓存工厂 factoryBean

只用一级缓存会导致，属性设置完成的Bean和未设置完成的Bean都保留在一级缓存中，可能导致空指针情况

只用两级缓存会导致，循环引用填充的对象可能并不是代理过的对象



