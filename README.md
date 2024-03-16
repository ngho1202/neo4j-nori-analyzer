# 정리 Notion URL
- [ENG] https://ngho1202.notion.site/How-to-add-nori-analyzer-tokenizer-on-Neo4j-6528b0617d854ac2a1bba15d821a13da
- [KOR] https://ngho1202.notion.site/Neo4j-nori-analyzer-tokenizer-aa63531439e646fdbda3c9e12ad4c8be?pvs=4

아래 글을 읽지 않고도, Neo4j-5.17.0 버전일 경우
>deploy/plugins 에 있는 neo4j-nori-analyzer-5.17.0.jar 파일과
deploy/import 에 있는 lucene-nori-analysis-9.8.0.jar 
>
파일을 neo4j Deploy 시에 추가하여주시면 됩니다!

# Neo4j Nori-Analyzer 플러그인 추가 방법

### 1. 사용하고 있는 Neo4j의 버전의 Lucene 버전 확인

- **[ 버전 파악 방법 ]**
    1. Neo4j 의 공식 Document 확인 
    2. 로컬 Docker에서 Exec 후 /var/lib/neo4j/lib 에서 확인
- **[ 버전 파악이 필요한 패키지 목록 ]**
    - **A. 사용하고 있는 Neo4j 버전**
    - **B. A에서 사용하고 있는 lucene 관련 패키지들의 버전**
        - 현재 Neo4j 에서 사용하고 있는 lucene 관련 패키지들은 다음의 4가지로 파악
            - lucene-core
            - **lucene-anlysis-common(중요!** 유사한 lucene-analyzers-common 과 다름)
            - lucene-backward-codecs
            - lucene-queryparser
                

### 2. 파악한 Neo4j와 Lucene 코드 `git clone` 후 필요한 부분만 jar 파일 생성
(본 repo의 submodule로 neo4j와 lucene을 갖고 있습니다. git submodule init, git submodule update 로 진행하여 주세요)

1. 2개를 레포를 **“1. 사용하고 있는 Neo4j의 버전의 Lucene 버전 확인”** 에서 파악한 버전으로 clone 및 branch 변환
2. `**lucene` repo에서는 아래 경로의 파일들을 복사하여 새로운 임의의 로컬 폴더에 복사**
    
    `lucene > lucene > analysis > nori > src 폴더` 
    
    src 폴더를 복사하여 local에 임의의 폴더생성(예시 : nori-analysis-lucene-9.8).
    
3. **`neo4j` repo에서는 아래 경로 폴더를 만들어 그 안에 NoriAnalyzer.java 파일 생성**
    
    1. (참고) neo4j 원본 repo의 analyzer/providers 경로
        
        `neo4j > community > fulltext-index > src > main > java > org > neo4j > kernel > api > impl > fulltext > analyzer > provieders > NoriAnalyzer.java`
        
    2. 원본(3-1.) 경로에서 src> 에 해당하는 부분에서부터 경로롤 폴더 생성 후 [NoriAnalyzer.java](http://NoriAnalyzer.java)(예시 화면에서는 Korean.java) 생성
        - [NoriAnalyzer.java](http://NoriAnalyzer.java) 파일
            
            ```java
            /*
             * Copyright (c) 2002-2019 "Neo4j,"
             * Neo4j Sweden AB [http://neo4j.com]
             *
             * This file is part of Neo4j.
             *
             * Neo4j is free software: you can redistribute it and/or modify
             * it under the terms of the GNU General Public License as published by
             * the Free Software Foundation, either version 3 of the License, or
             * (at your option) any later version.
             *
             * This program is distributed in the hope that it will be useful,
             * but WITHOUT ANY WARRANTY; without even the implied warranty of
             * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
             * GNU General Public License for more details.
             *
             * You should have received a copy of the GNU General Public License
             * along with this program.  If not, see <http://www.gnu.org/licenses/>.
             */
            package org.neo4j.kernel.api.impl.fulltext.analyzer.providers;
            
            import org.apache.lucene.analysis.Analyzer;
            import org.apache.lucene.analysis.ko.KoreanAnalyzer;
            import org.neo4j.annotations.service.ServiceProvider;
            import org.neo4j.graphdb.schema.AnalyzerProvider;
            
            @ServiceProvider
            public class Korean extends AnalyzerProvider
            {
                public Korean()
                {
                    super("nori");
                }
            
                @Override
                public Analyzer createAnalyzer()
                {
                    // 이부분에 개별적인 Parser 등 추가 Lucene 기능 추가
                    return new KoreanAnalyzer();
                }
            }
            
            ```
            
    
4. `**pom.xml` 작성**
    1. `pom.xml` 은 Python에서 `pyproject.toml` 또는 `requirements.txt` 에 해당하는 파일
        1. `pom.xml` - `mvn`
        2. `pyproject.toml` - `poetry`
        3. `requirements.txt` - `pip`
        
        이러한 관계라고 이해하면 됨
        
    2. pom.xml 작성 방법은 첨부된 Neo4j의 Custom Plugin 추가 방법을 준수
        
        [Setting up a plugin project - Java Reference](https://neo4j.com/docs/java-reference/current/extending-neo4j/project-setup/)
        
    - (참고) `lucene-nori-analysis` pom.xml 파일 > neo4j(5.17.0), lucene(9.8.0), java(17)
        
        ```xml
        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                              http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
        
          <groupId>org.apache.lucene</groupId>
          <artifactId>lucene-nori-analysis</artifactId>
          <version>9.8.0</version>
        
          <packaging>jar</packaging>
          <name>Neo4j Procedure Template</name>
          <description>A template project for building a Neo4j Procedure</description>
        
          <properties>
            <java.version>17</java.version>
            <neo4j.version>5.17.0</neo4j.version>
            <lucene.version>9.8.0</lucene.version>
        
            <maven.compiler.release>${java.version}</maven.compiler.release>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <neo4j-java-driver.version>${neo4j.version}</neo4j-java-driver.version>
            <junit-jupiter.version>5.10.0</junit-jupiter.version>
            <maven-shade-plugin.version>3.5.1</maven-shade-plugin.version>
            <maven-compiler-plugin.version>3.11.0</maven-compiler-plugin.version>
            <assertj.version>3.24.2</assertj.version>
            <maven-surefire-plugin.version>3.1.2</maven-surefire-plugin.version>
            <maven.version>3.9.4</maven.version>
            <maven-enforcer-plugin.version>3.4.1</maven-enforcer-plugin.version>
          </properties>
        
          <dependencies>
            <dependency>
              <!-- This gives us the Procedure API our runtime code uses.
                   We have a `provided` scope on it, because when this is
                   deployed in a Neo4j Instance, the API will be provided
                   by Neo4j. If you add non-Neo4j dependencies to this
                   project, their scope should normally be `compile` -->
              <groupId>org.neo4j</groupId>
              <artifactId>neo4j</artifactId>
              <version>${neo4j.version}</version>
              <scope>provided</scope>
            </dependency>
        
            <!-- Test Dependencies -->
           <dependency>
                <groupId>org.apache.lucene</groupId>
                <artifactId>lucene-core</artifactId>
                <version>${lucene.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.lucene</groupId>
                <artifactId>lucene-queryparser</artifactId>
                <version>${lucene.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.lucene</groupId>
                <artifactId>lucene-analysis-common</artifactId>
                <version>${lucene.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.lucene</groupId>
                <artifactId>lucene-backward-codecs</artifactId>
                <version>${lucene.version}</version>
            </dependency>
        
          </dependencies>
        
          <build>
            <plugins>
              <plugin>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>${maven-enforcer-plugin.version}</version>
                <executions>
                  <execution>
                    <id>enforce</id>
                    <goals>
                      <goal>enforce</goal>
                    </goals>
                    <phase>validate</phase>
                    <configuration>
                      <rules>
                        <requireJavaVersion>
                          <version>${java.version}</version>
                        </requireJavaVersion>
                        <requireMavenVersion>
                          <version>${maven.version}</version>
                        </requireMavenVersion>
                      </rules>
                    </configuration>
                  </execution>
                </executions>
              </plugin>
              <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.12.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
              </plugin>
              <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire-plugin.version}</version>
              </plugin>
              <plugin>
                <!-- This generates a jar-file with our procedure code,
                     plus any dependencies marked as `compile` scope.
                     This should then be deployed in the `plugins` directory
                     of each Neo4j instance in your deployment.
                     After a restart, the procedure is available for calling. -->
                <artifactId>maven-shade-plugin</artifactId>
                <version>${maven-shade-plugin.version}</version>
                <configuration>
                  <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                  <execution>
                    <phase>package</phase>
                    <goals>
                      <goal>shade</goal>
                    </goals>
                  </execution>
                </executions>
              </plugin>
            </plugins>
          </build>
        </project>
        ```
        
    - (참고) `neo4j-nori-analyzer` pom.xml 파일 > neo4j(5.17.0), lucene(9.8.0), java(17)
        
        ```xml
        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                              http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
        
          <groupId>org.neo4j</groupId>
          <artifactId>neo4j-nori-analyzer</artifactId>
          <version>5.17.0</version>
        
          <packaging>jar</packaging>
          <name>Neo4j Procedure Template</name>
          <description>A template project for building a Neo4j Procedure</description>
        
          <properties>
            <!-- Main Version Management -->
            <java.version>17</java.version>
            <neo4j.version>5.17.0</neo4j.version>
            <lucene.version>9.8.0</lucene.version>
        
            <maven.compiler.release>${java.version}</maven.compiler.release>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <neo4j-java-driver.version>${neo4j.version}</neo4j-java-driver.version>
            <junit-jupiter.version>5.10.0</junit-jupiter.version>
            <maven-shade-plugin.version>3.5.1</maven-shade-plugin.version>
            <maven-compiler-plugin.version>3.11.0</maven-compiler-plugin.version>
            <assertj.version>3.24.2</assertj.version>
            <maven-surefire-plugin.version>3.1.2</maven-surefire-plugin.version>
            <maven.version>3.9.4</maven.version>
            <maven-enforcer-plugin.version>3.4.1</maven-enforcer-plugin.version>
          </properties>
        
          <dependencies>
            <dependency>
              <!-- This gives us the Procedure API our runtime code uses.
                   We have a `provided` scope on it, because when this is
                   deployed in a Neo4j Instance, the API will be provided
                   by Neo4j. If you add non-Neo4j dependencies to this
                   project, their scope should normally be `compile` -->
              <groupId>org.neo4j</groupId>
              <artifactId>neo4j</artifactId>
              <version>${neo4j.version}</version>
              <scope>provided</scope>
            </dependency>
        
            <!-- Test Dependencies -->
           <dependency>
                <groupId>org.neo4j</groupId>
                <artifactId>neo4j-kernel</artifactId>
                <version>${neo4j.version}-SNAPSHOT</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.neo4j</groupId>
                <artifactId>neo4j-lucene-index</artifactId>
                <version>${neo4j.version}-SNAPSHOT</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.neo4j</groupId>
                <artifactId>neo4j-fulltext-index</artifactId>
                <version>${neo4j.version}-SNAPSHOT</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.lucene</groupId>
                <artifactId>lucene-analysis-nori</artifactId>
                <version>${lucene.version}</version>
                <scope>compile</scope>
            </dependency>
        
          </dependencies>
        
          <build>
            <plugins>
              <plugin>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>${maven-enforcer-plugin.version}</version>
                <executions>
                  <execution>
                    <id>enforce</id>
                    <goals>
                      <goal>enforce</goal>
                    </goals>
                    <phase>validate</phase>
                    <configuration>
                      <rules>
                        <requireJavaVersion>
                          <version>${java.version}</version>
                        </requireJavaVersion>
                        <requireMavenVersion>
                          <version>${maven.version}</version>
                        </requireMavenVersion>
                      </rules>
                    </configuration>
                  </execution>
                </executions>
              </plugin>
              <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.12.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
              </plugin>
              <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire-plugin.version}</version>
              </plugin>
              <plugin>
                <!-- This generates a jar-file with our procedure code,
                     plus any dependencies marked as `compile` scope.
                     This should then be deployed in the `plugins` directory
                     of each Neo4j instance in your deployment.
                     After a restart, the procedure is available for calling. -->
                <artifactId>maven-shade-plugin</artifactId>
                <version>${maven-shade-plugin.version}</version>
                <configuration>
                  <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                  <execution>
                    <phase>package</phase>
                    <goals>
                      <goal>shade</goal>
                    </goals>
                  </execution>
                </executions>
              </plugin>
            </plugins>
          </build>
        </project>
        ```
        

### 3. 생성한 jar 파일 Neo4j 플러그인에 추가

docker 기준 nori-analysis 는 /import 에 nori-analyzer 는 /plugin 에 추가


### 4. Command 실행순서(Neo4j 5.17.0 사용 시 그대로 사용 가능)
(clone 하지 않고 deploy/plugin 폴더와 deploy/import 폴더 안에 있는 jar파일을 deploy하는 neo4j에 추가하여도 됩니다!)
```
git clone <repo url>
git submodule init
git submodule update 
```
```
cd lucene-nori-analysis-9.8.0
mvn clean package
```
생성된 lucene-nori-analysis-9.8.0/target/ 하위에 lucene-nori-analysis-9.8.0.jar 파일을 Neo4j import 폴더에 추가 또는 복사
```
cd ..
cd neo4j-nori-analyzer-5.17.0
mvn clean package
```
생성된 neo4j-nori-analyzer-5.17.0/target/ 하위에 neo4j-nori-analyzer-5.17.0.jar 파일을 Neo4j plugin 폴더에 추가 또는 복사
