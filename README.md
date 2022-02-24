## Apache Hadoop 구축 및 Map-reduce 동작
### 1. 하둡 설치 요구사항 (Linux : Ubuntu, CentOS, Fedora, Redhat 등 다양한 OS를 사용가능하지만 본인 편한거 사용)
    - Java : 하둡은 Java 기반의 프레임워크로, java가 클러스터에 설치되어 있어야 함 
	          (java 8 혹은 11이 지원되지만 거의 모든 하둡버전에서 지원하는 자바 8(1.8)을 사용하는 것이 좋음
            
	- ssh : 하둡은 분산형 클러스터를 지원하고, 각 클러스터 간의 통신을 ssh 기반으로 진행하기 때문에 ssh 서비스가 필요
	          (대부분의 Linux OS가 설치 직후부터 사용가능하도록 지원하므로 `$ systemctl status sshd.service`와 같은 명령어로 작동을 확인)

### 2. 하둡 설치 및 설정 변경
	- 하둡은 아파치 미러 사이트에서 자신이 사용하고자 하는 버전의 Hadoop을 다운받은 (보통 tar.gz 확장자를 받으면 됨)
  
	- 압축을 풀고, hadoop이라는 최상위 디렉터리를 /etc 디렉터리 및으로 이동 (/etc/hadoop/... 경로가 되도록)
  
	- JAVA_HOME 경로 설정 : /etc/hadoop/hadoop-env.sh 파일에 "export JAVA_HOME=/자바/설치/경로(절대경로)"를 추가
  
	- 1차 점검 : /etc/hadoop 디렉터리 내에서 `$ bin/hadoop` 명령어 실행 -> 하둡 스크립트에 대한 설명이 출력되면 정상적으로 설치한 것

	- 하둡 모드에는 Standalone, Pseudo-Distributed, Fully-Distributed까지 총 3가지 모드가 존재하는데, 우선은 싱글 클러스터로
	  맵리듀스 동작을 체험해 볼 수 있는 Pseudo-Distributed 모드로 연습 [Standalone은 중간과정이 너무 생략되어 오히려 내부 로직을 이해하기 어렵다고 판단]
    
	- 하둡 설정 변경 : /etc/hadoop/core-site.xml | /etc/hadoop/hdfs-site.xml
		1. `core-site.xml`파일은 다음과 같이 내용을 수정
		<configuration>
    			<property>
        				<name>fs.defaultFS</name>
        				<value>hdfs://localhost:9000</value>
    			</property>
		</configuration>
		2. `hdfs-site.xml`파일은 다음과 같이 내용을 수정
		<configuration>
    			<property>
        				<name>dfs.replication</name>
        				<value>1</value>
    			</property>
		</configuration>
    
	- localhost로 인증없이 ssh 접속이 가능한지 확인 -> `$ ssh localhost` (되면 다음 ** 단계는 건너뜀)
  
	** 인증없이 ssh 접속이 가능하도록 설정
	  	$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  		$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  		$ chmod 0600 ~/.ssh/authorized_keys
	-> 위 과정은 ssh 키젠으로 ssh키를 설정하고, 비밀키를 클라이언트가 가지고 있어야 할 위치에 복사해두어 인증없이 ssh 접속 가능하게 만드는 과정

	- namenode 초기화(포맷) : `$ bin/hdfs namenode -format`
  
	- dfs(distributed file system) 실행 : `$ sbin/start-dfs.sh`
  
	- 2차 점검 : "http://localhost:9870/" 에 접속하여 어떠한 사이트가 보인다면 하둡 파일시스템 동작에 성공한 것

### 3. 하둡 맵리듀스 작업 수행
	- 하둡 분산 파일 시스템(HDFS)에 기본 디렉터리와 wordcount할 파일을 삽입
		$ bin/hdfs dfs -mkdir /user
		$ bin/hdfs dfs -mkdir /user/hadoopEx
		$ bin/hdfs dfs -mkdir input
		$ bin/hdfs dfs -put /etc/hadoop/*.xml input
    
	- 하둡 wordcount 예제 파일을 실행하여 input 디렉터리 내의 모든 파일의 wordcount를 수행
		$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar grep input output 'dfs[a-z.]+'
    
	- 맵리듀스 wordcount 작업이 수행된 결과는 output에 있으므로 이를 로컬로 가져와서 확인
		$ bin/hdfs dfs -get output output
		$ cat output/*
    
	- 혹은 위 과정을 hdfs 내에서 읽기 작업을 하는 것으로 수행할 수도 있음
		$ bin/hdfs dfs -cat output/*

### 4. 하둡 종료
	- 작업을 마치면 아래의 명령어로 하둡 데몬을 종료
		$ sbin/stop-dfs.sh
