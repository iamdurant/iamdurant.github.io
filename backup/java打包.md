```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.1.2</version> <!-- 使用合适的 Spring Boot 版本 -->
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal> <!-- 确保将所有依赖打包进 JAR -->
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```