# Intellij Idea导出jar包

## Java项目导出jar包

### 导出无依赖的jar

1. New Project

构造一个新的项目
![New Project](./java_new_project.png)

2. Create project from template

构造时选择模板，这个按需求来，不过一般选择一个，减少构造流程
![Create project from template](./java_create_project_from_template.png)

3. project folder

为新的项目设置名字、路劲、默认包名
![project folder](./java_project_folder.png)

4. Project Directory

新的Java项目的目录
![Project Directory](./java_project_directory.png)

5. New Library Directory

说明一点，Intellij Idea中的各个文件夹需要设置不同的功能，例如src就是存放代码的目录，而默认是没有存放jar的目录的，所以需要进行配置。

如下所示，src目录被设置为 Sources 类别
![Project Structure](./java_project_structure.png)

如下图，进入项目结构中，选择Module -> 选择新建的项目名 -> 选择Dependencies依赖 -> 点击+号 -> 选择JARS or directories弹出选择框
![New Library Directory](./java_lib_diretory.png)

6. Set Library Directory

为项目选择/设置jar的目录，点击OK，进入下一步
![Set Library Directory](./java_set_lib_directory.png)

7. Choose Categories of Selected Files

为刚才的文件选择合适的类别，这里指的就是 Jar Directory
![Choose Categories of Selected Files](./java_choose_categories_of_selected_files.png)

8. Write Code

写一个测试的Java文件
![Write Code](./java_write_code.png)

9. Set Artifacts

进行打包配置
![Set Artifacts](./java_set_artifacts.png)

10. Artifacts Configuration

- 选择Empty，出现下面的配置

![Artifacts Configuration](./java_java_artifacts_configuration_empty.png)

为Artifacts命名，这里设置和项目名相同的名字ExportJarDemo

![Artifacts Name](./java_artifacts_name_empty.png)

设置module output

![Module Output](./java_module_output.png)

选择Module Output的Module

![Choose Module](./java_choose_module.png)

点击OK, 出现下面的样子

![Module Output pic](./java_module_output_png.png)

- 选择 From modules with dependencies

![From modules with dependencies](java_from modules_with_dependencies.png)

点击OK, 出现如下图

![From modules with dependencies pic](java_module_output.png)

11. Build Project

先编译项目，生成class文件

![Build Project](./java_build_project.png)

编译生成的class文件

![Build Project Pic](./java_build_project_pic.png)

12. Build Artifacts

进行打包

![Build Artifacts](./java_build_artifacts.png)

确认

![Build Artifacts Confirm](java_build_artifacts_confirm.png)

编译生成的Jar文件

![Artifacts pic](./java_artifacts_png.png)

13. Test

测试生成的jar包

![Test](./java_test.png)


### 导出有依赖的jar

1. dependencies

如下图所示，引入gson-2.8.0.jar

![Demo Dependencies](./java_demo_dependencies.png)

2. export jar

和上面的步骤是一样的，导出jar, 如下是新建的Artifacts

![Artifacts Output Dependencies](./java_artifacts_output_dependencies.png)

3. Test

如下是生成的jar包结构

![Jar Dependencies](./java_jar_denpendencies.png)

可以从中发现，导出的jar包包括了依赖的jar包

测试如下

![Test Dependencies](./java_test_dependencies.png)

### 导入有依赖的jar, 不包括依赖的代码

1. Artifacts Configuration

如下图所示，导出的jar(这里的名字是ExportJarDemo)下有两部分，一部分是"Extracted 'gson-2.8.0.jar", 一部分是'ExportJarDemo' compile output, 分别表示依赖的jar包和 ExportJarDemo 的编译输出。

我们只希望我们导出的jar中包含本项目的编译输出，那么只需要在Artifacts的配置中删除导入的jar, 如下所示，选择依赖的jar, 点击-号

![Artifacts Output Without JAR Dependencies](./java_artifacts_output_without_jar_dependencies.png)

点击OK, 其他操作相同

2. Test

如下图所示，是生成的jar包结构

![JAR without JAR Dependencies](./java_jar_without_jar_dependencies.png)

### 导出可运行的jar

1. Write Code

添加可执行的代码，如下所示，添加了一个包含了main方法的类

![Demo Excuted](./java_demo_excuted.png)

2. Create Artifacts

配置Artifacts, 如下所示，在创建Artifacts的时候，选择 Main Class

![Artifacts Creation Excuted](./java_artifacts_creation_excuted.png)

点击OK, 然后Build Artifacts, 生成的jar包结构如下图

![JAR Excuted](./java_jar_excuted.png)

3. Test

直接运行该jar包

![Test Excuted](./java_test_excuted.png)

### 导出可运行的jar, 不包括依赖的代码

和上面```导入有依赖的jar, 不包括依赖的代码```操作是一样的，只需要在Artifacts的配置中删除导入的jar即可，不再赘述。




## gradle Java项目导出jar包

### 导出无依赖的jar

1. New Project

创建gradle的java项目

![Gradle New Project](./gradle_new_project.png)

2. Project Configuration

进行项目 Group Id, Artifacts, Version 配置

![Gradle Project Configuration](./gradle_project_configuartion.png)

3. Project Creation Configuration

进行项目创建的一些配置，如下图

![Gradle Project Creation Configuration](./gradle_project_creation_configuration.png)

4. Project folder

设置项目名和路径

![Gradle Project Folder](./gradle_project_folder.png)

生成的项目结构如图所示

![Gradle Project Directory](./gradle_project_directory.png)

5. Export JAR

如果没有使用gradle依赖其他jar包，那么可以直接使用上面的方式导出jar, 所以没有什么特殊之处

### 有依赖的jar导出

1. gradle dependencies

使用gradle导入jar包

![Build Gradle](./gradle_build_gradle.png)

2. Export JAR

虽然使用的gradle构建项目，但是仍然可以使用上面的方式导出jar

如下图，配置Artifacts, 选择Module, 这里选择 GradleDemo_main 

![gradle JAR dependencies](./gradle_jar_denpendencies.png)

选择OK

![Gradle JAR Dependencies](./gradle_jar_dependencies.png)

如果需要在导出的jar中包含依赖的jar, 则保留这样，如果不想在导出的jar中含有依赖的jar, 则移除导出的jar

### 使用gradle命令进行打包











































<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
