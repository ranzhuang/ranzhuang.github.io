---
title: Maven创建Web工程以及SSM框架目录结构的搭建
date: 2018-10-17
tags: Java
---

### 创建Web工程

1、选择"File"--->"New"--->"Project"，如图所示：

![图1](maven创建web工程/1.png)

2、在弹出的视图左侧选择"Maven"，右侧选择SDK，并勾选上"Create from archetype"选择框，选择"org.apache.maven.archetypes:maven-archetype-webapp"，如图所示：

![图2](maven创建web工程/2.png)

3、在视图中输入GroupId、ArtifactId的名称，如图所示：

![图3](maven创建web工程/3.png)

4、视图中主要包含Maven的路径，用户设置文件的路径、repository的路径，可以不做修改，直接点击"Next"

![图4](maven创建web工程/4.png)

5、 在弹出的视图中可以设置工程名字、工程的路径等信息，修改完成点击"Finish"，完成创建

![图5](maven创建web工程/5.png)

Maven所创建的Web工程目录结构图展示:

![图6](maven创建web工程/6.png)

### 查看并修改工程结构

点击"File"--->选择"Project Structure..."弹出工程结构视图，选择左侧的"Modules"标签，如图进行设置

![图7](maven创建web工程/7.png)

选择左侧的"Artifacts"标签，如图进行设置

![图8](maven创建web工程/8.png)

### 设置Tomcat

点击"Run"菜单项--->点击"Edit Configurations..."弹出配置视图，点击视图左上角的"+"添加本地的Tomcat，如图所示

![图9](maven创建web工程/9.png)

配置Tomcat Server，主要是配置Tomcat Server的名字和服务器的版本，如图所示

![图10](maven创建web工程/10.png)

部署项目，依然是在配置视图中，选择"Deployment"标签，如图进行工程部署

![图11](maven创建web工程/11.png)


>运行项目，在网页中出现"Hello World!"字样，项目就部署成功了


### SSM框架目录结构搭建

在"图6"中展示了Maven创建Web工程的目录结构，发现太过于简单，需要创建更多的路径以及包，使项目看起来更加的清晰，更加有利于维护，下图将展示一个较为简单的SSM目录结构

![图12](maven创建web工程/12.png)

在idea中，点击"File"--->选择"Project Structure..."，可在Project Structure视图中的"Modules"标签中去进行文件夹的标记，如图所示

![图13](maven创建web工程/13.png)



>写在最后
>
>在idea开发maven项目的时候，如果resources路径配置不正确，可能会导致无法找到xml配置文件，从而无法成功注入bean。此时，可以在pom.xml里的"build"标签中添加如下代码：


	<resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </resource>
    </resources>
