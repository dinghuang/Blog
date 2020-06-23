---
title: Selenium测试框架
date: 2019-01-04 09:47:00
tags:
    - JAVA
categories: JAVA
---
      
[Selenium官网](https://www.seleniumhq.org/)

# Selenium简介
Selenium可以对浏览器进行自动化测试。它主要用于自动化Web应用程序以进行测试，但当然不仅限于此。无聊的基于Web的管理任务也可以自动化。 Selenium得到了一些最大的浏览器供应商的支持，这些供应商已采取（或正在采取）将Selenium作为其浏览器本机部分的步骤。它也是无数其他浏览器自动化工具，API和框架的核心技术。支持多种语言，在java中可以作为自动化测试框架，在python中可以模拟页面用户点击对自动化爬虫进行补充。

支持的浏览器：
![image](https://minios.strongsickcat.com/dinghuang-blog-picture/007yGiDRgy1fyy2e6qz72j310m0wu45r.jpg)
# Selenium使用
Selenium提供了一种非常简单的开发方式，例如用Chrome开发的话，去开发者工具下载Katalon Selenium IDE，如图所示。
![image](https://minios.strongsickcat.com/dinghuang-blog-picture/007yGiDRgy1fyy2ex4m1gj315o15gjwi.jpg)

## 开始录制
录制过程中，IDE会自动帮我们把命令行插入到测试用例中，包括：
- 单击链接
- 输入值
- 从下拉框选择数据
- 单击按钮或者选择框
点击开始录制，在最上方输入网站域名，后期可以通过更换域名来实现不同域名下的应用的测试。

## 使用上下文菜单添加验证和断言
使用Selenium IDE录制，转到显示测试应用程序的浏览器，然后右键单击页面上的任意位置。您将看到一个显示验证和/或断言命令的上下文菜单。
Selenium命令有三种“风格”：动作，访问器和断言。
- 动作是通常操纵应用程序状态的命令。他们执行“点击此链接”和“选择该选项”之类的操作。如果操作失败或出错，则停止执行当前测试。
- 访问者检查应用程序的状态并将结果存储在变量中，例如“storeTitle”。它们还用于自动生成断言。
- 断言与访问器类似，但它们验证应用程序的状态是否符合预期。示例包括“确保页面标题为X”和“验证是否选中此复选框”。

## 脚本语法

命令很简单，由2个参数构成：

verifyText | //div//a[2] |Login
---|---|---
这些参数并不总是必需的;这取决于命令。在某些情况下，两者都是必需的，在其他情况下需要一个参数，而在另一些情况下，命令可能根本不需要参数。这里有几个例子：

chooseCancelOnNextPrompt| | |
---|---|---
pause | 500 |
type | id=phone|	(555) 666-7066
type| id=address1|${myVariableAddress}
命令参考描述了每个命令的参数要求。 参数有所不同，但它们通常是：
- Locators用于标识页面内UI元素的定位器。
- text patterns用于验证或声明预期页面内容的文本模式。
-  text patterns or selenium variables文本模式或selenium变量，用于在输入字段中输入文本或从选项列表中选择选项。
### 常用的Selenium命令
- **open** 打开url页面
- **click** 执行单击操作，并可选择等待加载新页面。
- **verifyTitle/assertTitle** 验证预期的页面标题。
- **verifyTextPresent** 验证预期文本是否在页面上的某个位置。
- **verifyText** 验证预期文本及其相应的HTML标记出现在页面上。
- **verifyTable** 验证表的预期内容。

### 验证页面元素
#### 断言与验证的选择
- **assert** 错误后会不继续执行并中断当前的测试用例
- **verify** 错误后会继续执行
的最佳用途是对测试命令进行逻辑分组，并使用“assert”后跟一个或多个“verify”测试命令启动每个组。一个例子如下：

##### verifyElementPresent

Command | Target | Value
---|--- |---
verifyElementPresent | //div/p/img | 

此命令验证页面上是否存在由``<img>`` HTML标记的存在指定的图像，并且它遵循``<div>``标记和``<p>``标记。第一个（也是唯一的）参数是一个定位器，用于告诉Selenese命令如何查找元素。
``verifyElementPresent``可用于检查页面中是否存在任何HTML标记。您可以检查链接，段落，分区``<div>``等是否存在。以下是一些示例。

Command | Target | Value
---|--- |---
verifyElementPresent | //div/p | 
verifyElementPresent | //div/a | 
verifyElementPresent | id=Login | 
verifyElementPresent | 	link=Go to Marketing Research | 
verifyElementPresent | 	//a[2] | 
verifyElementPresent |	//head/title | 

##### verifyText
必须在测试文本及其UI元素时使用verifyText。 verifyText必须使用定位器。如果选择XPath或DOM定位器，则可以验证特定文本是否显示在页面上相对于页面上其他UI组件的特定位置。
Command | Target | Value
---|--- |---
verifyText |//table/tr/td/div/p | This is my text and it occurs right after the div inside the table.
#### 定位元素
对于许多Selenium命令，需要一个目标。此目标标识Web应用程序内容中的元素，并包含位置策略，后跟位置格式为locatorType = location。在许多情况下可以省略定位器类型。下面解释各种定位器类型，每个定位器类型都有示例。
##### 按标识符定位
例如，页面源可以具有id和name属性，如下所示：
```
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
   <input name="continue" type="button" value="Clear" />
  </form>
 </body>
<html>
```
以下定位器策略将返回上面由行号指示的HTML片段中的元素：
- identifier=loginForm (3)
- identifier=password (5)
- identifier=continue (6)
- continue (6)
由于定位器的标识符类型是默认值，因此上面的前三个示例中的标识符=不是必需的。

**通过id定位**

 id=loginForm (3)
 
 **通过名称定位**
 
- name=username (4)
- name=continue value=Clear (7)
- name=continue Clear (7)
- name=continue type=button (7)

与某些类型的XPath和DOM定位器不同，上面三种类型的定位器允许Selenium测试UI元素，而与其在页面上的位置无关。因此，如果页面结构和组织被更改，测试仍将通过。您可能想也可能不想测试页面结构是否发生变化。在Web设计者经常更改页面但其功能必须经过回归测试的情况下，通过id和name属性进行测试，或者通过任何HTML属性进行测试变得非常重要。

由于只有xpath定位符以“//”开头，因此在指定XPath定位符时不必包含xpath =标签。

- xpath=/html/body/form[1] (3) - 绝对路径（如果HTML仅稍微更改，则会中断）
- //form[1] (3) - HTML中的第一个表单元素
- xpath=//form[@id='loginForm'] (3) - 表单元素，其属性名为“id”，值为“loginForm”
- xpath=//form[input/@name='username'] (3) - 带有输入子元素的第一个表单元素，其属性名为“name”，值为“username”
- //input[@name='username'] (4) - 第一个输入元素，其属性名为“name”，值为“username”
- //form[@id='loginForm']/input[1] (4) - 表单元素的第一个输入子元素，其属性名为“id”，值为“loginForm”
- //input[@name='continue'][@type='button'] (7) -输入名为'name'的属性和值'continue'以及名为'type'的属性和值'button'
- //form[@id='loginForm']/input[4] (7) - 表单元素的第四个输入子元素，其属性名为“id”，值为“loginForm”


主要的语法参考[Xpath](https://www.w3.org/TR/2017/REC-xpath-31-20170321/)
可以使用浏览器的devtools复制XPath：
![image](https://www.seleniumhq.org/docs/_images/chapt2_img18_Copy_xpath.png)

**通过链接文本查找超链接**

这是一种使用链接文本在网页中查找超链接的简单方法。如果存在具有相同文本的两个链接，则将使用第一个匹配。
```
<html>
 <body>
  <p>Are you sure you want to do this?</p>
  <a href="continue.html">Continue</a>
  <a href="cancel.html">Cancel</a>
</body>
<html>
```

- link=Continue (4)
- link=Cancel (5)

**通过CSS定位**

```
 <html>
  <body>
   <form id="loginForm">
    <input class="required" name="username" type="text" />
    <input class="required passfield" name="password" type="password" />
    <input name="continue" type="submit" value="Login" />
    <input name="continue" type="button" value="Clear" />
   </form>
 </body>
 <html>
```
- css=form#loginForm (3)
- css=input[name="username"] (4)
- css=input.required[type="text"] (4)
- css=input.passfield (5)
- css=#loginForm input[type="button"] (7)
- css=#loginForm input:nth-child(2) (5)

可以参考[ the W3C publication](http://www.w3.org/TR/css3-selectors/)

没有明确的设定选择器的话，将会默认使用id选择器

### 存储命令和Selenium变量
可以使用Selenium变量在脚本开头存储常量。此外，当与数据驱动的测试设计（在后面的部分中讨论）结合使用时，Selenium变量可用于存储从命令行，从另一个程序或从文件传递到测试程序的值。

plain store命令是许多存储命令中最基本的命令，可用于在selenium变量中简单地存储常量值。它需要两个参数，即要存储的文本值和一个selenium变量。在为变量选择名称时，请使用仅包含字母数字字符的标准变量命名约定。

Command| Target|Value |
---|---|---
store | 	paul@mysite.org |userName

稍后在脚本中，将需要使用变量的存储值。要访问变量的值，请将变量括在大括号（{}）中，并在其前面加上美元符号。

Command| Target|Value |
---|---|---
verifyText | 	//div/p |${userName}

变量的常见用途是存储输入字段的输入

Command| Target|Value |
---|---|---
type | 	id=login |${userName}

**storeText**

StoreText对应于verifyText。它使用定位器来标识特定的页面文本。如果找到该文本，则存储在变量中。 StoreText可用于从正在测试的页面中提取文本。

echo命令可以用来打印变量

### Alerts, Popups, and Multiple Windows

```
<!DOCTYPE HTML>
<html>
<head>
  <script type="text/javascript">
    function output(resultText){
      document.getElementById('output').childNodes[0].nodeValue=resultText;
    }

    function show_confirm(){
      var confirmation=confirm("Chose an option.");
      if (confirmation==true){
        output("Confirmed.");
      }
      else{
        output("Rejected!");
      }
    }

    function show_alert(){
      alert("I'm blocking!");
      output("Alert is gone.");
    }
    function show_prompt(){
      var response = prompt("What's the best web QA tool?","Selenium");
      output(response);
    }
    function open_window(windowName){
      window.open("newWindow.html",windowName);
    }
    </script>
</head>
<body>

  <input type="button" id="btnConfirm" onclick="show_confirm()" value="Show confirm box" />
  <input type="button" id="btnAlert" onclick="show_alert()" value="Show alert" />
  <input type="button" id="btnPrompt" onclick="show_prompt()" value="Show prompt" />
  <a href="newWindow.html" id="lnkNewWindow" target="_blank">New Window Link</a>
  <input type="button" id="btnNewNamelessWindow" onclick="open_window()" value="Open Nameless Window" />
  <input type="button" id="btnNewNamedWindow" onclick="open_window('Mike')" value="Open Named Window" />

  <br />
  <span id="output">
  </span>
</body>
</html>
```


Command| Description
---|---
assertFoo(pattern) |如果模式与弹出窗口的文本不匹配，则抛出错误
assertFooPresent |如果弹出窗口不可用则抛出错误
assertFooNotPresent |如果存在任何弹出窗口则抛出错误
storeFoo(variable) |将弹出文本存储在变量中
storeFooPresent(variable)|将弹出窗口的文本存储在变量中并返回true或false

在Selenium下运行时，不会显示JavaScript弹出窗口。这是因为函数调用实际上是由Selenium自己的JavaScript在运行时覆盖的。但是，仅仅因为你看不到弹出窗口并不意味着你不必处理它。要处理弹出窗口，必须调用其assertFoo（模式）函数。如果您未能断言是否存在弹出窗口，则您的下一个命令将被阻止，您将收到类似于以下错误的错误[错误]错误
``error] Error: There was an unexpected Confirmation! [Chose an option.]``

#### Alerts

让我们从Alerts开始，因为它们是最简单的弹出窗口。首先，在浏览器中打开上面的HTML示例，然后单击“Show alert”按钮。您会注意到，在您关闭警报后，页面上会显示“警报已消失。”文本。现在使用Selenium IDE录制完成相同的步骤，并在关闭警报后验证是否添加了文本。您的测试看起来像这样：

Command| Target|value
---|---|---
open |/|
click |btnAlert|
assertAlert |I’m blocking!  |
verifyTextPresent |Alert is gone.  |
您可能会想“这很奇怪，我从未试图断言该警报。”但这是Selenium-IDE处理并为您关闭警报。如果您删除该步骤并重播测试，您将获得以下内容 ``[error] Error: There was an unexpected Alert! [I'm blocking!].``

如果您只想声明警报存在但是不知道或不关心它包含哪个文本，则可以使用assertAlertPresent。这将返回true或false，错误地停止测试。

#### Confirmations

确认的行为与警报的行为大致相同，其中assertConfirmation和assertConfirmationPresent提供与其警报对应物相同的特征。但是，默认情况下，Selenium会在弹出确认时选择“确定”。尝试单击示例页面中的“显示确认框”按钮，但单击弹出窗口中的“取消”按钮，然后断言输出文本。您的测试可能如下所示：

Command| Target|value
---|---|---
open |/|
click |btnAlert|
chooseCancelOnNextConfirmation | |
assertConfirmation |	Choose an option.  |
verifyTextPresent |Rejected |

chooseCancelOnNextConfirmation函数告诉Selenium所有后续确认都应该返回false。可以通过调用chooseOkOnNextConfirmation来重置它。

可能会注意到您无法重播此测试，因为Selenium抱怨存在未经处理的确认。这是因为Selenium-IDE记录的事件顺序导致click和chooseCancelOnNextConfirmation被置于错误的顺序（如果考虑它就有意义，Selenium在打开确认之前无法知道正在取消）切换这两个命令，你的测试运行正常。

#### Prompts

提示的行为与警报的行为大致相同，其中assertPrompt和assertPromptPresent提供与其警报对应项相同的特征。默认情况下，Selenium会在弹出提示时等待您输入数据。尝试单击示例页面中的“显示提示”按钮，然后在提示中输入“Selenium”。测试可能如下所示：

Command| Target|value
---|---|---
open |/|
answerOnNextPrompt |Selenium!|
click |	id=btnPrompt |
assertPrompt |		What’s the best web QA tool? |
verifyTextPresent |	Selenium! |

如果在提示中选择取消，您可能会注意到answerOnNextPrompt只显示空白目标。 Selenium对取消和提示上的空白条目基本上是一样的。

### 调试
#### 断点
要设置断点，请选择一个命令，单击鼠标右键，然后从上下文菜单中选择“切换断点”。然后单击“运行”按钮以从开始到断点运行测试用例。
从测试用例的中间位置到结束位置运行测试用例或者到达起始点之后的断点有时也很有用。例如，假设您的测试用例首先登录到网站，然后执行一系列测试，并且您正在尝试调试其中一个测试。但是，您只需要登录一次，但是在开发测试时需要不断重新运行测试。您可以登录一次，然后从测试用例的登录部分之后的起点运行测试用例。这将阻止您每次重新运行测试用例时都必须手动注销。
#### 按步骤执行测试
要一次执行一个测试用例（“step through”），只需重复按此按钮。

![image](https://www.seleniumhq.org/docs/_images/chapt2_img09_Step.png)
#### Find Button
“查找”按钮用于查看当前显示的网页（在浏览器中）中当前选定的Selenium命令中使用的UI元素。在为命令的第一个参数构建定位器时，这非常有用（请参阅定位元素一节）。它可以与标识网页上UI元素的任何命令一起使用，例如，单击，单击和等待，键入，以及某些断言和验证命令等。 从表视图中，选择具有locator参数的任何命令。单击“查找”按钮。现在查看网页：应该有一个明亮的绿色矩形，包围locator参数指定的元素。

## Java中应用Selenium


[项目地址](https://github.com/dinghuang/seleniumJavaTest)

我在本地录制了一个简单的脚本，选择导出到java+junit，类似下面：
```
public class SearchGoogle {
    private WebDriver driver;
    private boolean acceptNextAlert = true;
    private StringBuffer verificationErrors = new StringBuffer();
    private SeleniumConfigure seleniumConfigure = SeleniumConfigureParse.getSeleniumConfigure();

    @Before
    public void setUp() {
        driver = DriverUtil.getDriver();
        driver .manage().window().maximize();//全屏
        driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);
    }

    @Test
    public void testSearchGoogle() {
        driver.get(seleniumConfigure.getBaseUrl());
        driver.findElement(By.name("q")).click();
        driver.findElement(By.name("q")).clear();
        driver.findElement(By.name("q")).sendKeys("google");
        driver.findElement(By.name("q")).sendKeys(Keys.ENTER);
    }

    @After
    public void tearDown() {
        driver.quit();
        String verificationErrorString = verificationErrors.toString();
        if (!"".equals(verificationErrorString)) {
            fail(verificationErrorString);
        }
    }

    private boolean isElementPresent(By by) {
        try {
            driver.findElement(by);
            return true;
        } catch (NoSuchElementException e) {
            return false;
        }
    }

    private boolean isAlertPresent() {
        try {
            driver.switchTo().alert();
            return true;
        } catch (NoAlertPresentException e) {
            return false;
        }
    }

    private String closeAlertAndGetItsText() {
        try {
            Alert alert = driver.switchTo().alert();
            String alertText = alert.getText();
            if (acceptNextAlert) {
                alert.accept();
            } else {
                alert.dismiss();
            }
            return alertText;
        } finally {
            acceptNextAlert = true;
        }
    }
}

```

新建项目，pom.xml如下
```
 <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>choerodon</groupId>
    <artifactId>selenium</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-server</artifactId>
            <version>3.14.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.yaml/snakeyaml -->
        <dependency>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
            <version>1.23</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <!--<dependency>-->
            <!--<groupId>org.projectlombok</groupId>-->
            <!--<artifactId>lombok</artifactId>-->
            <!--<version>1.18.4</version>-->
            <!--<scope>provided</scope>-->
        <!--</dependency>-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.5</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M3</version>
                <dependencies>
                    <dependency>
                        <groupId>org.apache.maven.surefire</groupId>
                        <artifactId>surefire-junit47</artifactId>
                        <version>3.0.0-M3</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <outputDirectory>${basedir}/output</outputDirectory>
                    <outputName>测试报告</outputName>
                </configuration>
            </plugin>
            <!--<plugin>-->
                <!--<groupId>org.apache.maven.plugins</groupId>-->
                <!--<artifactId>maven-site-plugin</artifactId>-->
                <!--<version>2.1</version>-->
                <!--<configuration>-->
                    <!--<outputDirectory>${basedir}/output</outputDirectory>-->
                    <!--<outputName>测试报告</outputName>-->
                <!--</configuration>-->
            <!--</plugin>-->
        </plugins>
    </build>

</project>
 ```
 
 ## 与Docker结合
 
 使用远程的驱动服务来测试，目前只支持Chrome和FireFox，本地如果要起服务，请在docker中执行下面的命令启动服务
 ```
 docker pull elgalu/selenium
 docker run -d --name=grid -p 4444:24444 -p 5900:25900  -e TZ="Asia/Shanghai" -e MAX_INSTANCES=20 -e MAX_SESSIONS=20  -v /Users/dinghuang/Documents/Tool/selenium/shm:/dev/shm --privileged elgalu/selenium
 docker exec grid wait_all_done 30s
 ```
 可以在``http://localhost:4444/grid/console``中查看详情
 
 关闭服务命令：
 ```
 docker exec grid stop
 docker stop grid
 ```
 在JAVA代码中，可以通过远程的docker容器启动浏览器进行测试
 ```
 webDriver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), browser);
 ```

 
 
 
 