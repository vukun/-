＃打包部署到服务器的操作（主要针对多模块互相依赖的项目）：

1、首先对父模块引入打包的方式，在pom.xml文件中操作。本项目的父模块是maven工程(主要是统一本项目中所有用到的jar包等依赖)，所以需要打包成pom文件
    <groupId>com.njupt.gmall</groupId>
    <artifactId>gmall-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>  //添加打包的方式是为pom
    
2、其次在父模块中需要引入依赖父模块的所有的子模块
    <modules>
        <module>../gmall-api</module>
        <module>../gmall-cart-web</module>
       ...
    </modules>
    
3、此时、针对于父模块的操作已完成。
4、然后对所有的工具模块(子模块，没有启动类的模块)进行操作：首先，在子模块的pom.xml中，将子模块所依赖的父模块引入到<parent></parent>标签中
    <parent>
        <groupId>com.njupt.gmall</groupId>
        <artifactId>gmall-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../gmall-parent</relativePath>  //引入父模块的路径
    </parent>
    
   然后设置本子模块的打包方式：war包方式。
        <groupId>com.njupt.gmall</groupId>
        <artifactId>gmall-common-util</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>war</packaging>   //本模块的打包方式
    
  此时如果本模块，是一个springboot的module的话，是需要把内置的tomcat排除掉。即pom.xml文件中如果有这个依赖的话
        <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
  需要在依赖中加入(主要是用来排除内置的tomcat，交给外置的tomcat运行)：
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
5、此时，所有的工具模块也操作完成。
6、对含有启动类的模块进行操作：此项操作步骤只需要在上面第4个步骤完成后，在src目录下，在启动类的同级目录下，添加一个SpringBootStartApplication类，它继承了SpringBootServletInitializer类，并且重写了configure()方法：

   public class SpringBootStartApplication extends SpringBootServletInitializer {

         @Override
         protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
              return builder.sources(GmallManagerServiceApplication.class);
         }
   }
   
7、此时，还需要把模块中所用到的http://...地址改成服务器的地址。至此，含有启动类的模块也操作完成。
8、进行正式的打包操作：
       在idea中点击右侧的maven管理，首先对某一个要打包的模块进行clean操作，然后再双击package，此时在该模块的文件目录下会产生一个target目录，打包好的war包就在该目录下。
       此时，打包的操作也已经完成。
9、进行正式的部署操作：
      直接将打包好的war包放到服务器的tomcat的webapps的目录下，启动tomcat即可。
10、此时浏览器需要访问http:// ip地址:对应的端口号/war包的名称，才能访问到该模块工程。
    可以在tomcat的conf文件夹的server.xml文件中的：
    <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
     后添加一行，作用是将该访问路径设置为根目录下。    
    <Context path="/" docBase="/opt/tomcat/tomcat_10/webapps/gmall-passport-web" debug="0" reloadable="false"/>
11、至此，所有的打包部署操作都已完成。  
    
