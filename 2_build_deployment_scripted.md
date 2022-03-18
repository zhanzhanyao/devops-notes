## 第二部分： 实现部署流水线
### 第六章：构建与部署脚本化
#### 对构建部署的理解
- 在大型或分布式团队里，使用脚本执行应用程序构建，测试和打包
- 向测试环境和生产环境进行自动化部署需要一系列步骤： 配置应用程序，初始化数据，配置基础设施、操作系统和中间件，以及安装所需的模拟外部系统。
- 通过构建工具执行部署会遇到一些麻烦：目标环境和所以中间件对于部属机制会有一些约束。
#### 本章的内容
了解使用构建和部署工具的原则，给出实际工作所需的基础知识
- 构建工具概览
- 构建部署脚本化的原则与实践
- 基于JVM的应用程序项目
- 部署脚本化
- Tips

#### 2. 构建工具概览
- 构建工具的核心功能：对依赖关系建模
- 在执行过程中，以正确的顺序执行一系列任务，计算如何达到指定目标，且被依赖的任务只需要运行一次。
- 每个任务都有两点内容：它是做什么的，它依赖于什么。 
- DSL domain-specific language  
  ![deployment pipeline](image/dependancy_of_testing.png)
- 构建工具不同点：
  - 任务导向：依据任务描述依赖网络
  - 产品导向：根据生成产物描述依赖网络。将状态以时间戳的形式保存在每个任务执行后生成的文件中。 Make
- 流行的构建工具：Make， SCons， Ant， NAnt， MSBuild， Maven， Rake， Buildr， Psake  

#### 3. 构建部署脚本化的原则与实践

##### 3.1 为部署流水线的每个阶段创建脚本
- 确保所有脚本在版本控制库中，最好开发与运维人员共同完成构建脚本和部署脚本
- 当项目开始时，可以将部署流水线的每个操作都放在同一脚本，即使尚未自动化的步骤，也可有对应得哑操作。
- 脚本过长时，让部署流水线得每个阶段使用单独的脚本：
  - 提交阶段的脚本： 完成编译、打包、运行提交测试套件、执行代码静态分析
  - 功能验收测试脚本调用部署工具，将应用程序部署到适当环境，并准备相关数据，之后运行验收测试
  - 运行非功能测试得脚本，例如压力测试，安全测试

##### 3.2 使用恰当的技术部署应用程序
- 部署流水线中，提交阶段的后续如自动化验收测试，用户验收测试都需要将应用程序部署到类生产环境中
- 自动化部署时应使用恰当的工机，而不是通用脚本语言

##### 3.3 使用相同得脚本向所有环境部署
- 使用同样的流程部署应用程序到每个环境，能确保构建和部署流程经过有效测试
- 针对不同环境的配置信息不同，将配置信息从脚本中分离出来，保存到版本控制系统，采用一些机制让部署脚本去获取

##### 3.4 使用操作系统自带的包管理工具
- 书中的“二进制包”指部署过程中需要放到目标环境中的所有内容，是构建过程中产生的一堆文件，以及应用程序所需的库文件，还包括版本库中的某些静态文件

##### 3.5 确保部署流程是幂等的
- 部署：无论开始时目标环境处于何种状态， 部署流程总是令目标环境达到相同的正确的状态
- 完成以上的方法是每次都以自动化或者虚拟化将良好的基线环境作为起点。基线环境指所有需要的中间件，让应用环境正常工作的软硬件。
- 部署前验证对环境的假设是否成立，例如中间件是否安装，是否运行，版本是否正确，应用程序所依赖的外部服务是否正在运行。
- 每次部署都要以版本库中的某个版本的二进制包开始
- 使用效果幂等的工具进行部署？

##### 3.6 部署系统的增量式演进

#### 4. 面向JVM的应用程序的项目结构   

    /project-name
      README.txt
      LICENSE/txt
      /src
        /main
        /java    java source code for your project
        /scala   if you use other language, they go at the same level
        /resources    ressources for your project   
        /filters    resources filter files   
        /assembly    assembly descriptors       
        /config    configration files   
        /webapp    web application resources
      /test
        /java    test sources
        /resources    test resources
        /filters    test resource filters
      /site    source for your project website
      /doc    any other documentation
    /lib
      /runtime    libraries your project needs at run time
      /test    libraries required to run tests
      /build    libraries required to build your project

##### 4.1 源代码管理
- 遵循标准的Java实践，将文件放在以包名为目录名的目录中，每个文件保存一个类，若不遵守可能会引入难以发现的缺陷
- 遵守Java的命名习惯
- 生成的配置和元数据不应放在src，应该放在target目录中，不应提交到版本控制系统

##### 4.2 测试管理
- 将测试的源代码放在test/java目录中
- 单元测试放在与包名对应的目录中，某个类的测试应该与类放在同一包中

##### 4.3 构建输出的管理
- Maven构建时会把生成的代码，元数据放在target文件，将此目录删除就能清除前一次构建结果
- 构建最终生成的JAR WAR EAR二进制包，存在target目录中
- 最开始一个项目一个JAR，项目复杂后可能不同组件不同JAR包， 创建多个JAR文件的目的是令应用程序的部署更简单，令构建流程更高效
  

    /project-name
      /target
        /classes    complied classes
        /test-classes    complied test classes
        /surefire-reports   test report

##### 4.4 库文件管理
- 依赖库的两种管理方式
- 确保应用程序所依赖的库文件都和应用程序的二进制包在一起打包