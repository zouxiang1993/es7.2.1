ES代码是用gradle管理的，这里将其转换成用maven管理的项目 

1. 下载源代码 git clone https://github.com/elastic/elasticsearch.git, 并切换到v7.2.1 tag 
2. 新建maven工程, 新建server模块, 将server模块下的所有代码复制到idea中, 复制过程中会有一些类中的引用包名被改变，
导致后面编译报错。
3.  将build.gradle中的外部依赖添加到pom.xml中，其中的版本号引用在buildSrc\version.properties文件中
4. 编译, 这时会报错。有一些类中的Bucket指代不明, 可以参照git clone下来的源文件修改。   
    还会报找不到类 : ExtendedPluginsClassLoader,   
    这个类在libs\plugin-classloader\src\main\java\org\elasticsearch\plugins目录下
5. 新建一个目录D:\elasticsearch-env\test-env, 在下面再新建modules目录, 放入需要的modules；  
    再新建一个config目录, 放入 log4j2.properties 和 elasticsearch.yml 配置文件
    再新建一个data目录和logs目录
6. 添加 VM Options : 
    ```text 
    -Des.path.home=D:\elasticsearch-env\test-env 
    -Des.path.conf=D:\elasticsearch-env\test-env\config 
    -Dlog4j2.disable.jmx=true
    ```
7. 在jdk的conf\security\java.policy文件末尾添加几行: 
    ```text
	permission java.lang.RuntimePermission "createClassLoader"; 
	permission java.lang.RuntimePermission "getClassLoader"; 
	permission java.lang.RuntimePermission "accessDeclaredMembers";	
    ```
8. 运行org.elasticsearch.bootstrap.Elasticsearch中的main方法
 