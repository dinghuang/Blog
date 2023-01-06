---
title: Spock-Spring-Gradle 单元测试
date: 2022-6-8 15:47:00
tags:
    - JAVA
categories: JAVA
---

# Spock介绍
``Spock``是国外一款优秀的测试框架，基于``BDD``（行为驱动开发）思想实现，功能非常强大。``Spock``结合``Groovy``动态语言的特点，提供了各种标签，并采用简单、通用、结构化的描述语言，让编写测试代码更加简洁、高效。``Spock``作为测试框架，在开发效率、可读性和维护性方面均取得了不错的收益。

# 修改项目配置
项目用``gradle``管理，用的是``7.4.2``版本，``spock``用的是``2.0-M3-groovy-3.0``版本。

## 配置修改

修改文件``build.gradle``并添加依赖
```
List spockTest = [
        "org.spockframework:spock-core:2.0-M3-groovy-3.0",
        "org.spockframework:spock-spring:2.0-M3-groovy-3.0",
        "org.mockito:mockito-core:4.3.1",
        "org.mockito:mockito-inline:4.3.1",
        "org.springframework.boot:spring-boot-starter-test"
]

dependencies {
    implementation 'org.codehaus.groovy:groovy:3.0.5'
    testImplementation spockTest
}

sourceSets {
    main {
        java { srcDirs = ['src/main/java'] }
        resources { srcDirs = ['src/main/resources'] }
    }

    test {
        java { srcDirs = ['src/test/groovy'] }
        resources { srcDirs = ['src/test/resources'] }
    }
}

apply from: 'test.gradle'
```
修改``test.gradle``，这步主要是和``jacoco``结合生成报告（带覆盖率，用的``jacoco``的``offline``特性）
```
apply from: 'testFilesConf.gradle'
apply plugin: 'java'
apply plugin: "groovy"
apply plugin: "jacoco"

jacoco {
    toolVersion = "0.8.7"
    reportsDir = file("$buildDir/customJacocoReportDir")
}

configurations {
    jacocoAnt
    jacocoRuntime
}

test {
    useJUnitPlatform()
    maxHeapSize = "1G"
    jacoco {
        destinationFile = file("$buildDir/jacoco/test.exec")
        classDumpDir = file("$buildDir/jacoco/classpathdumps")
    }
    testLogging {
        afterSuite { desc, result ->
            if (!desc.parent) {
                println "Unit Tests: ${result.resultType} (${result.testCount} tests, ${result.successfulTestCount} successes, ${result.failedTestCount} failures, ${result.skippedTestCount} skipped)"
            }
        }
    }
}

def excludedSourcesPattern = ['**/entity/**', '**/dto/**']

jacocoTestReport {
    reports {
        html.enabled true
        csv.enabled false
        xml.enabled true
        xml.destination file("$buildDir/jacocoHtml/jacocoXml.xml")
        html.destination file("$buildDir/reports/jacoco/test/html")
    }
    afterEvaluate {
        getClassDirectories().setFrom(
                classDirectories.files.collect {
                    fileTree(dir: it, excludes: excludedSourcesPattern)
                }
        )
    }
}

def compileClassPath = [  "${project.buildDir}/classes/java/main"]

task instrument(dependsOn: ['classes']) {
    ext.outputDir = buildDir.path + '/intermediates/classes-instrumented/Java'
    doLast {
        ant.taskdef(name: 'instrument',
                classname: 'org.jacoco.ant.InstrumentTask',
                classpath: configurations.jacocoAnt.asPath)
        ant.instrument(destdir: outputDir) {
            compileClassPath.each {
                fileset(dir: it)
            }
        }
    }
}

gradle.taskGraph.whenReady { graph ->
    if (graph.hasTask(instrument)) {
        tasks.withType(Test) {
            doFirst {
                systemProperty 'jacoco-agent.destfile', buildDir.path + '/jacoco/test.exec'
                classpath = files(instrument.outputDir) + classpath + configurations.jacocoRuntime
            }
        }
    }
}

task jacocoReport(dependsOn: ['instrument', 'test']) {
    doLast {
        ant.taskdef(name: 'report',
                classname: 'org.jacoco.ant.ReportTask',
                classpath: configurations.jacocoAnt.asPath)
        ant.report() {
            executiondata {
                ant.file(file: buildDir.path + "/jacoco/test.exec")
            }
            structure(name: project.name) {
                classfiles {
                    compileClassPath.each {
                        fileset(dir: it)
                    }
                }
                sourcefiles {
                    fileset(dir: 'src/main/java')
                }
            }
            html(destdir: buildDir.path + '/reports/jacoco')
        }
    }
}


check.dependsOn jacocoTestReport
```
## 对Spring的封装
这边对Spring做了一个封装，可以做成一个测试的maven模块，先写一个自定义注解，主要是用作Spring容器的启动，同时制定启动的配置文件为test
```

/**
 * @version 1.0
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/5/30 16:03
 * @Modify By: dinghuang
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootTest(webEnvironment = RANDOM_PORT, classes = ServerApplication.class)
@ActiveProfiles("test")
@Stepwise
public @interface CustomTest {

}

```
这个时候我们在test的resource下面的``application.properties``修改一下配置
```
spring.profiles.active=test
```

接下来就是写代码了


# 代码编写
代码写在``test``目录下，跟``junit``不一样的是，目录结构是：``test->groovy->xxxx.xxx.xxx``

> 需要``IDEA``把``groovy``的``mark directory``设置成``test root``


## 单元测试
这边不懂Spock的语法的，可以百度下，这边直接略过


先看业务代码，业务很简单，查一个字典表返回数据
```
/**
 * @Version 1.0
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/4/1 18:05
 * @Modify By: dinghuang
 */
@Service
public class SysDictService {

    private static final Logger LOGGER = LoggerFactory.getLogger(SysDictService.class);

    @Autowired
    private SysDictDao sysDictDao;

    public String getDictName(String dict) {
        SysDict sysDict = sysDictDao.selectByPrimaryKey(dict);
        return sysDict.getDictname();
    }


}
```

### Mock模拟
```

/**
 *
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/5/30 16:18
 * @version 1.0
 * @Modify By: dinghuang
 *
 */
@CustomTest
class TestControllerTest extends Specification {

    @Autowired
    SysDictTestService sysDictTestService

    @SpringBean
    SysDictDao sysDictDao = Mock()

    def 'Mock测试'() {
        given:
        def dict = '系统'
        1 * sysDictDao.selectByPrimaryKey(dict) >> new SysDict('系统')

        when:
        def result = sysDictTestService.getDictName(dict)

        then:
        result == '系统'

    }

}

```

### Where使用
正常我们单元测试率有个分支覆盖率指标，如果用junit写太麻烦了，用spock可以非常简洁，假如字典的逻辑不同的dictName有不同的处理逻辑
```

/**
 *
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/5/30 16:18
 * @version 1.0
 * @Modify By: dinghuang
 *
 */
@CustomTest
class TestControllerTest extends Specification {

    @Autowired
    SysDictTestService sysDictTestService

    @SpringBean
    SysDictDao sysDictDao = Mock()

    def 'Where测试'() {
        given:
        sysDictDao.selectByPrimaryKey(a) >> d

        when:
        def b = sysDictTestService.getDictName(a)

        then:
        b == c

        where:
        a       | d                       || c
        '10000' | new SysDict('系统')       || '系统'
        '10004' | new SysDict('多点登录处理方式') || '多点登录处理方式'

    }

}

```
这里可以用``@Unroll``注解，可以把每一次调用作为一个单独的测试用例运行，这样运行后的单元测试结果更加直观：
```
@Unroll
def 'Where测试'() {
    given:
    sysDictDao.selectByPrimaryKey(a) >> d

    when:
    def b = sysDictTestService.getDictName(a)

    then:
    b == c

    where:
    a       | d                       || c
    '10000' | new SysDict('系统')       || '系统'
    '10004' | new SysDict('多点登录处理方式') || '多点登录处理方式'

}
```

### 测试异常
先修改一下业务代码，如下
```

/**
 * @Version 1.0
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/4/1 18:05
 * @Modify By: dinghuang
 */
@Service
public class SysDictTestService {

    private static final Logger LOGGER = LoggerFactory.getLogger(SysDictTestService.class);

    @Autowired
    private SysDictDao sysDictDao;

    public String getDictName(String dict) {
        if ("测试异常".equals(dict)) {
            throw new BusinessException(ErrorCodeEnum.SYS_ERROR);
        }
        SysDict sysDict = sysDictDao.selectByPrimaryKey(dict);
        return sysDict.getDictname();
    }


}
```
写异常测试类
```

/**
 *
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/5/30 16:18
 * @version 1.0
 * @Modify By: dinghuang
 *
 */
@CustomTest
class TestControllerTest extends Specification {

    @Autowired
    SysDictTestService sysDictTestService

    def '测试异常'() {
        given:
        def dict = '测试异常'

        when:
        sysDictTestService.getDictName(dict)

        then:
        def exception = thrown(BusinessException)
        exception.errorCode == '21379999'
        exception.message == 'bizErrCode=21379999,bizErrMsg=系统异常'

    }

}

```

### 静态方法Mock
先修改一下业务代码
```

/**
 * @Version 1.0
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/4/1 18:05
 * @Modify By: dinghuang
 */
@Service
public class SysDictTestService {

    private static final Logger LOGGER = LoggerFactory.getLogger(SysDictTestService.class);

    @Autowired
    private SysDictDao sysDictDao;

    public String getDateStr() {
        return DateUtil.getCurrentSysDate();
    }

}

```
```
public class DateUtil {
     public static String getCurrentSysDate() {
        return getDate(new Date(), yyyyMMdd_PATTERN);
    }
}
```
静态方法测试
```

/**
 *
 * @Author: dinghuang
 * @Description:
 * @Date: since 2022/5/30 16:18
 * @version 1.0
 * @Modify By: dinghuang
 *
 */
@CustomTest
class TestControllerTest extends Specification {

    @Autowired
    SysDictTestService sysDictTestService

    @Shared
    MockedStatic<DateUtil> dateUtilMockedStatic

    void setup() {
        dateUtilMockedStatic = Mockito.mockStatic(DateUtil.class)
        // mock静态类
    }

    def '测试异常'() {
        given:
        Mockito.when(DateUtil.getCurrentSysDate()).thenReturn("1991-01-02")

        when:
        def date = sysDictTestService.getDateStr()

        then:
        date == "1991-01-02"
    }

}

```
### 测试sql
这个原理是在每个单元测试可以写一个前置后置处理方法，比如跑测试A，会自动跑测试A的前置，把数据库初始化进去，数据初始化进去，用的是H2内存数据库，但是这个东西对特定数据库的一些语法不支持，所以不太建议这么做，简单的sql的确可以测试出问题

```
/**
* 直接获取待测试的mapper
*/
def personInfoMapper = MapperUtil.getMapper(PersonInfoMapper.class)

/**
* 测试数据准备，通常为sql表结构创建用的ddl，支持多个文件以逗号分隔。
*/
def setup() {
executeSqlScriptFile("com/xxx/xxx/xxx/......../schema.sql")
}
/**
* 数据表清除，通常待drop的数据表
*/
def cleanup() {
dropTables("person_info")
}

/**
* 直接构造数据库中的数据表,此方法适用于数据量较小的mapper sql测试
*/
@MyDbUnit(
    content = {
        person_info(id: 1, name: "abc", age: 21)
        person_info(id: 2, name: "bcd", age: 22)
        person_info(id: 3, name: "cde", age: 23)
    }
)
def "demo1_01"() {
when:
int beforeCount = personInfoMapper.count()
// groovy sql用于快速执行sql，不仅能验证数据结果，也可向数据中添加数据。
def result = new Sql(dataSource).firstRow("select * from `person_info`") 
int deleteCount = personInfoMapper.deleteById(1L)
int afterCount = personInfoMapper.count()

then:
beforeCount == 3
result.name == "abc"
deleteCount == 1
afterCount == 2
}
```

### 报告
正常运行完成``gradle test``会生成报告在路径``build\reports\tests\test\index.html``，这个是html的，没有覆盖率的情况，如图所示：

![](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG758.png)


运行我们``test.gradle``中自己写的task，``gradle jacocoReport``，会生成单元测试覆盖报告在路径``build\reports\jacoco\index.html``，如图所示

![](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG759.png)