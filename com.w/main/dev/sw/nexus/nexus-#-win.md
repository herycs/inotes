# nexus

> jar包依赖管理软件

# 1.安装

- windows下

  - 解压下载的压缩包

  - 桌面右键左下角的win图标，选择管理员类别命令行打开

  - cd到解压文件的bin目录下

  - 键入命令：.\nexus.bat install

    > 卸载：.\nexus.bat uninstall
    >
    > 启动：.\nexus.bat start

# 2.配置文件

## 2.1主配置文件

- conf下的nexus.properties

  ```properties
  #
  # Sonatype Nexus (TM) Open Source Version
  # Copyright (c) 2008-present Sonatype, Inc.
  # All rights reserved. Includes the third-party code listed at http://links.sonatype.com/products/nexus/oss/attributions.
  #
  # This program and the accompanying materials are made available under the terms of the Eclipse Public License Version 1.0,
  # which accompanies this distribution and is available at http://www.eclipse.org/legal/epl-v10.html.
  #
  # Sonatype Nexus (TM) Professional Version is available from Sonatype, Inc. "Sonatype" and "Sonatype Nexus" are trademarks
  # of Sonatype, Inc. Apache Maven is a trademark of the Apache Software Foundation. M2eclipse is a trademark of the
  # Eclipse Foundation. All other trademarks are the property of their respective owners.
  #
  
  # Sonatype Nexus
  # ==============
  # This is the most basic configuration of Nexus.
  
  # Jetty section
  #下面的是默认端口号（针对图形化界面），若你的8081已被占用，建议修改
  application-port=8081
  application-host=0.0.0.0
  nexus-webapp=${bundleBasedir}/nexus
  nexus-webapp-context-path=/nexus
  
  # Nexus section
  nexus-work=${bundleBasedir}/../sonatype-work/nexus
  runtime=${bundleBasedir}/nexus/WEB-INF
  ```

  

# 3.访问图形化界面

- 浏览器访问：localhost:8088/nexus

  > 访问成功后路径会变为：http://localhost:8081/nexus/#welcome，同时出现图形化界面

- 登录：

  username：admin， password：admin123

