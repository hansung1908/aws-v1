# AWS 배포용 프로젝트

### Springboot 2.6.3, JDK 11
- devtools
- springweb
- lombok

### 배포 위치 EC2

### 배포 방법
- 로컬에서 github 업로드
- EC2에서 github 다운로드
##### 1. 자바 설치(jdk)
- sudo apt update
- sudo apt install openjdk-11-jdk

##### 2. github 배포용 프로젝트 다운로드
- git clone https://github.com/hansung1908/aws-v1.git

##### 3. gradlew 실행 권한 부여
- cd ~/aws-v1
- chmod u+x gradlew

##### 4. gradlew로 프로젝트를 jar로 변경
- ./gradlew build

##### 5. java로 jar를 실행 (x)
- cd build/libs
- java -jar *.jar

##### 6. nohup 명령어
- 리눅스에서 프로세스를 실행한 터미널의 세션이 끊어지더라도 지속적으로 동작할 수 있게 해주는 명령어
- cd aws-v1 -> cd build -> cd libs -> nohup java -jar *.jar
- 포그라운드에서 실행하면 터미널 종료시 그대로 종료되므로 명령어 끝에 &을 붙혀서 백그라운드에서 실행
- 자동으로 로그를 남김 nohup.out
- 로그파일을 mylog.out으로 변경하려면 nohup java -jar *.jar > mylog.out &
- 에러 파일 - 2 - nohup.out, 표준 출력 - 1 - nohup.out
- 각 로그 파일을 변경하려면 nohup java -jar *.jar 1>log.out 2>err.out &
- 이유 : 스크립트를 통한 자동화 시 표준 출력과 오류를 분리하기 위해

##### 7. 시간 변경
- 모든 지역의 시간대 리스트를 볼려면 timedatectl list-timezones
- 서울 시간대로 변경하려면 sudo timedatectl set-timezone Asia/Seoul
- 표준 출력 모니터로 출력하려면 echo "안녕"

##### 8. pid 찾기
- ps -ef | grep *.jar | grep -v grep | awk '{print $2}'
- pgrep -f *.jar
- 스프링 종료를 위한 스크립트 작성 vi spring-stop.sh
- i를 눌러 입력 모드 후 밑에 내용을 작성
- echo "Springboot Stop....."
- SPRING_PID=$(pgrep -f *.jar)
- echo $SPRING_PID
- kill -9 $SPRING_PID
- esc로 입력 모드 해제 후 :wq로 저장 후 종료
- 해당 파일의 실행권한을 주기 위해 chomod u+x spring-stop.sh
- 실행은 ./spring-stop.sh
- 변수는 대문자로 작성
- $변수 - 변수의 값을 실행 혹 출력
- $(명령어) - 명령어의 결과를 리턴

##### 9. cron
- 부하, 에러 등으로 서버 종료 시 자동으로 재시작 할 수 있게 해주는 명령어
- crontab -e
- 2번 vim.basic 선택 -> i(insert)
- #* * * * * ls -l 1>cron.log (#은 제외)
- 분(0-59) 시간(0-23) 일(1-31) 월(1-12) 요일(0-7)
- #* 3,4 * * *(새벽 3시와 4시 실행), * 3-6 * * *(새벽 3시부터 6시까지 실행)(#은 제외)
- vi로 myScript.sh 생성 후 입력
- crontab -l 1>crontab_new
- echo "* * * * * /home/ubuntu/job.sh" 1>>crontab_new
- crontab crontab_new
- * * * * * /home/ubuntu/job.sh을 crontab_new에 넣고 crontab을 통해 주기적인 실행
- job.sh 생성 -> ls -l > /home/ubuntu/cron.log
- cron.log에 현재 폴더 결과를 추가
- myScript.sh 실행 권한 준 후 실행

### 고민 1
- 로컬에서 EC2와 같은 환경을 만들어서 프로젝트를 테스트하고 빌드해서 배포할 수 없을까?
- Docker 가상화 기술

### 고민 2
- 로컬이 아니더라도 EC2와 같은 환경에서 테스트를 한번 해보고 잘 작동하면 배포할 수 없을까?
- 테스트를 위한 서버

### 고민 3
- 사용자가 폭증하면 서버가 멈추지 않을까? 자동 오토 스케일링 하는 방법이 없을까?
- 엘라스틱 빈스톡

### 고민 4
- github에 푸시만 해도 자동으로 배포되는 서비스가 없을까? 
- github action
