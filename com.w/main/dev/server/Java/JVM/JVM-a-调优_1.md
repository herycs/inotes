# JVM - a - 调优



* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* HotSpot参数分类

    > 标准： - 开头，所有的HotSpot都支持
    >
    > 非标准：-X 开头，特定版本HotSpot支持特定命令
    >
    > 不稳定：-XX 开头，下个版本可能取消

    java -version

    java -X

    java -XX:+PrintFlagsWithComments //只有debug版本能用

java -XX:+PrintCommandLineFlags HelloGC

1. java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC  HelloGC
    PrintGCDetails PrintGCTimeStamps PrintGCCauses
2. java -XX:+UseConcMarkSweepGC -XX:+PrintCommandLineFlags HelloGC
3. java -XX:+PrintFlagsInitial 默认参数值
4. java -XX:+PrintFlagsFinal 最终参数值
5. java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
6. java -XX:+PrintFlagsFinal -version |grep GCjava -XX:+PrintFlagsFinal -version | wc -l 
    共728个参数