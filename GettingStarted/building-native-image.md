# Quarkus - 构建本地可执行程序

原文链接 : [https://quarkus.io/guides/building-native-image](https://quarkus.io/guides/building-native-image)

本教程包括 :

- 将应用编译为本地可执行程序.
- 将本地可执行程序打包进入容器.
- debug本地可执行程序.

## GraalVM

构建一个本地可执行文件需要使用GraalVM的发行版。GraalMV有三个发行版:

- Oracle GraalVM社区版（CE）
- Oracle GraalVM企业版（EE）
- Mandrel。

Oracle和Mandrel发行版之间的区别如下 :

- Mandrel是Oracle GraalVM CE的一个下游发行版。Mandrel的主要目标是提供一种方法来构建专门为支持Quarkus而设计的本地可执行文件。

- Mandrel发行版的代码库来自于上游的Oracle GraalVM CE代码库，只做了一些小的改动，但也有一些重要的排除项，对于Quarkus本地应用程序来说是不必要的。它们支持与Oracle GraalVM CE相同的构建本地可执行文件的能力，在功能上没有重大变化。值得注意的是，它们不包括对多语言编程的支持。之所以排除这些功能，是为了给大多数Quarkus用户提供更好的支持水平。与Oracle GraalVM CE/EE相比，这些不包括的内容也意味着Mandrel的发行量大大减少。

- Mandrel的构建方式与Oracle GraalVM CE略有不同，它使用的是标准的OpenJDK项目。这意味着它不能从Oracle对用于构建他们自己的GraalVM下载的OpenJDK版本添加的一些小的增强功能中获益。这些增强功能被省略了，因为上游的OpenJDK并不管理它们，也无法担保。在涉及到一致性和安全性时，这一点尤其重要。

- Mandrel目前只被推荐用于构建针对Linux容器化环境的本地可执行文件。这意味着，Mandrel用户应该使用容器来构建他们的本地可执行文件。如果你要为macOS或Windows目标平台构建本地可执行文件，你应该考虑使用Oracle GraalVM，因为Mandrel目前不针对这些平台。在裸机Linux上直接构建本地可执行文件是可能的，详情可参见Mandrel README和Mandrel版本。
