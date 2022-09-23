### 1. Hive 다운 및 압축 해제하기

1) wget https://downloads.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz
2) tar xzf apache-hive-3.1.3-bin.tar.gz
ls 로 압축 해제 완료를 확인하자

—

### 2. Hive 환경 변수 설정하기(bashrc)

1) **nano .bashrc**
.bashrc의 맨 밑에 아래의 문구를 추가시켜주고, ctrl+x -> y -> enter로 저장한다.
```bash
export HIVE_HOME=“/home/hdoop/apache-hive-3.1.3-bin”
export PATH=$PATH:$HIVE_HOME/bin
```
2) **source ~/.bashrc**

—
### 3. hive-config.sh 파일 편집하기

- **Hive는 HDFS와 상호작용을 해야된다.

1) **nano $HIVE_HOME/bin/hive-config.sh**
2) **export HADOOP_HOME=/home/hadoop/hadoop-3.2.1**
> HADOOP_HOME 의 경우 nano .bashrc 안에 적힌 위치와 동일하게 입력하자
![hive-config.sh](Scala 언어 학습/asset/c855cde357f02fb123bab3591de55e2f.png)
저장하고 나옴

—
### 4. HDFS에 Hive directory 만들기

- hdfs layer에 데이터를 저장하기 위해 분리된 두 개의 디렉토리를 생성
- (tmp 와 다른 하나 메인)
- HDFS를 사용하므로 hadoop 먼저실행 **(start-dfs.sh)**
![start-dfs.sh](Scala 언어 학습/asset/afc3194325f83717508b39a36324441b.png)
**1) tmp directory 생성**
-tmp directory는 Hive process의 중간 데이터 결과를 저장하는 용도로 사용하자

1-1) `hdfs dfs -mkdir /tmp`

1-2) `hdfs dfs -chmod g+w /tmp`
쓰기 실행 권한 부여

1-3) `hdfs dfs -ls /`
권한 추가되었는지 확인
![-ls](Scala 언어 학습/asset/9b49276d453cd82c76684d9642c47c18.png)


**2) warehouse directory 생성**
2-1) `hdfs dfs -mkdir -p /user/hive/warehouse`
2-2) `hdfs dfs -chmod g+w /user/hive/warehouse`
2-3) `hdfs dfs -ls /user/hive`

—
### 5. hive-site.xml 파일 설정하기

1) `cd $HIVE_HOME/conf`
2) `cp hive-default.xml.template hive-site.xml`
cp a b : a를 b 이름으로 복제본 만듦
3) `nano hive-site.xml`
-property 로 시작되는 부분을 찾아 아래의 두 개의 property를 추가시키자.
```xml
<property>
    <name>system:jave.io.tmpdir</name>
    <value>/tmp/hive/java</value>
</property>

<property>
    <name>system:user.name</name>
    <value>${user.name}</value>
</property>
```
그 이후 `Ctrl + w` 키로 검색창을 켜고 `javax.jdo.option.ConnectionURL` 을 검색하여 property를 찾고 value 를 `jdbc:derby:/home/hdoop/apache-hive3.1.3-bin/metastore\_db;databaseName=metastore\_db;create=true`로 수정해주자

—
### 6. derby database 시작하기
`$HIVE_HOME/bin/schematool -initSchema -dbType derby` 을 실행해야하는데 오류가 날 것이다. (java.lang.NoSuchMethodError)
hadoop ↔ hive 간 guava version 호환성 문제를 해결하러 가자

1) `ls $HIVE_HOME/lib`
![hive guava version](8abb72878389dfab58eb14e49777f9ce.png)

hive의 guava version 은 19.0

2) `ls $HADOOP_HOME/share/hadoop/hdfs/lib`
![hadoop guava version](Scala 언어 학습/asset/54cb368c5cc42d3c9d0463fc98ef76c6.png)

hadoop의 guava version 은 27.0

3) `rm $HIVE_HOME/lib/guava-19.0jar`
- hive의 guava 삭제
4) `cp $HADOOP_HOME/share/hadoop/hdfs/lib/guava-27.0-jre.jar $HIVE_HOME/lib/`
- hadoop의 guava를 hive에 복사

다시 `$HIVE_HOME/bin/schematool -initSchema -dbType derby`
또 에러 발생! line 3223, column96 에 이상한 문자가 있다

`cd $HIVE_HOME/conf` 로 이동해서 
`nano hive-site.xml` 켜고
`ctrl + shift + -` 입력하면 행,열 이동가능 `3223,96` 입력해서 특수기호 들어간 것들 지워주자.
 `$HIVE_HOME/bin/schematool -initSchema -dbType derby` 재시도.
 
 —
 ### 7. HIVE Client Shell 시작하기
 1) `cd $HIVE_HOME/bin`
 2) `hive`
 ![최종](Scala 언어 학습/asset/0ac8397d442c2a6691f5a893cea0c2f3.png)
 

—
### 부록 hql 파일 생성 및 실행
1) `touch 이름.hql`
2) `nano 이름.hql` or 파일 직접 들어가서 작성
3) `hive -f 이름.hql` 하면 실행 됨

#### hive 명령어 아무데서 나 막 치면 안 됨.

**HiveException java.lang.RuntimeException: Unable to instantiate**
오류뜰 때 팁.
```bash
rm -rf metastore_db/
schematool -initSchema -dbType derby
```

- start-dfs..sh 이후에 바로 `hive` 명령어 X / 하둡 실행되는 동안 초기 몇 초는 safe mode 돌입됨.

hdfs 의 문서를 hive 에 가져오는 방법
```sql
DROP TABLE IF EXISTS MULGA;

CREATE EXTERNAL TABLE IF NOT EXISTS MULGA (
a DATE,
b STRING,
c INT,
d STRING,
e STRING,
f INT,
g STRING,
h INT,
i INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/hive/mulga'
tblproperties ("skip.header.line.count"="1");

DROP TABLE IF EXISTS TEST;

CREATE EXTERNAL TABLE IF NOT EXISTS TEST (
a DATE,
b STRING,
c INT,
d STRING,
e STRING,
f INT,
g STRING,
h INT,
i INT
)
STORED AS ORC
LOCATION '/user/hive/test';

INSERT OVERWRITE TABLE TEST
SELECT * FROM MULGA;
```
  
```sql
export table 테이블이름 to '폴더';
```



- Hive 3.0 이상 부터 Mapreduce 를 권장하지 않는다 spark / tez 로 변경을 추천하지만 tez 환경설정 도중 classpath 가 오류가 나는지 hive가 작동을 하지 않는다… 이후 계속 고민해 봐야할 부분 (버전마다, 블로그마다 )


### mysql 연동에 관한 고찰

- **hive-site.xml**

```xml
<configuration>
    <property>   
        <name>hive.metastore.local</name>   
        <value>false</value>   
    </property>   
    <property>   
        <name>hive.metastore.warehouse.dir</name>
        <value>hdfs://master:9000/user/hive/warehouse</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>hive</value>
    </property>
</configuration>
```
connectionURL 예시 :    ` jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true`
##### 접속하기
- `mysql -u root -p`
- 비밀번호 입력은 root 계정 비밀번호
- 들어와서 `CREATE USER '생성할 id'@'%' IDENTIFIED BY '비밀번호';`
- `GRANT all on *.* to '생성할 id'@'%';`
- `flush privileges;`

`schematool -dbType mysql -initSchema`

`schematool -initSchema -dbType mysql -userName ssafy -passWord 'BigDataDDP77!' -verbose`


##### JDBC Program
Given below is the JDBC program to apply the Group By clause for the given example
```java

```