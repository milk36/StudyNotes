javafx疑难问题
---
1. JDK11 配置
    
    * [JDK11使用IDEA，配置JavaFX](https://codeantenna.com/a/EV62cW9bLp)
    
      ```
      //idea run configuration
      --module-path "D:\Program Files\Java\javafx-sdk-11.0.2\lib"
      --add-modules javafx.controls,javafx.fxml
      ```
    * 命令行中的运行方式, 如:
    
      `java -jar --module-path "D:\Program Files\Java\javafx-sdk-11.0.2\lib" --add-modules javafx.controls,javafx.fxml  ChatClient-0.0.1.jar`
    
    * [gradle 配置方式](https://openjfx.io/openjfx-docs/#gradle)
    
      [javafx-gradle-plugin -- github](https://github.com/openjfx/javafx-gradle-plugin)

      ```gradle
      plugins{
          id 'application'
          id 'org.openjfx.javafxplugin' version '0.0.9'
      }

      javafx {
          version = "11"
          modules = [ 'javafx.controls', 'javafx.fxml' ]
      }

      mainClassName= 'milk.chat.client.App'

      jar {
          manifest {
              attributes 'Main-Class': mainClassName
          }
          from {
              configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
          }
      }

      ```