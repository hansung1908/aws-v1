# AWS 배포용 프로젝트

### stack
- Springboot
- JDK
- devtools
- springweb
- lombok

### 배포 위치 
- EC2

### 배포 방법
- 로컬에서 github 업로드
- EC2에서 github 다운로드

##### 0. 환경설정
- 이미 깔려있는데 처음부터 다시 하려면 aws-v1폴더를 통째로 삭제후 배포
- rm -r /home/ubuntu/aws-v1
- 스크립트로 삭제할 경우 물어보는 부분 때문에 스크립트가 작동을 안함, 그래서 rm -rf 사

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

##### 6. 시간 변경
- 모든 지역의 시간대 리스트를 볼려면 timedatectl list-timezones
- 서울 시간대로 변경하려면 sudo timedatectl set-timezone Asia/Seoul
- 표준 출력 모니터로 출력하려면 echo "안녕"

##### 7. nohup 명령어
- 리눅스에서 프로세스를 실행한 터미널의 세션이 끊어지더라도 지속적으로 동작할 수 있게 해주는 명령어
- cd aws-v1 -> cd build -> cd libs -> nohup java -jar *.jar
- 포그라운드에서 실행하면 터미널 종료시 그대로 종료되므로 명령어 끝에 &을 붙혀서 백그라운드에서 실행
- 자동으로 로그를 남김 nohup.out
- 로그파일을 mylog.out으로 변경하려면 nohup java -jar *.jar > mylog.out &
- 에러 파일 - 2 - nohup.out, 표준 출력 - 1 - nohup.out
- 각 로그 파일을 변경하려면 nohup java -jar *.jar 1>log.out 2>err.out &
- 이유 : 스크립트를 통한 자동화 시 표준 출력과 오류를 분리하기 위함

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
- /* * * * * ls -l 1>cron.log (/은 제외)
- 분(0-59) 시간(0-23) 일(1-31) 월(1-12) 요일(0-7)
- #* 3,4 * * *(새벽 3시와 4시 실행), * 3-6 * * *(새벽 3시부터 6시까지 실행)(#은 제외)
- vi로 myScript.sh 생성 후 입력
- /# 크론탭 내용을 crontab_new 파일로 옮긴다. (/은 제외)
- crontab -l 1>crontab_new
- /# crontab_new 파일에 echo의 내용을 추가한다. (/은 제외)
- echo "* * * * * /home/ubuntu/job.sh" 1>>crontab_new
- /# crontab에 crontab_new에 작성한 내용을 반영한다. (/은 제외)
- crontab crontab_new
- /# crontab_new 파일은 삭제한다. (/은 제외)
- rm crontab_new
- /* * * * * /home/ubuntu/job.sh을 crontab_new에 넣고 crontab을 통해 주기적인 실행 (/은 제외)
- job.sh 생성 -> ls -l >> /home/ubuntu/cron.log
- cron.log에 현재 폴더 결과를 추가
- myScript.sh 실행 권한 준 후 실행

##### 10. 재배포 스크립트 생성
- 스프링 서버 재시작을 위한 파일 생성
- mkdir cron_restart
- 서버를 중단하기 위한 스크립트 생성 vi sprint-stop.sh
- echo "SPRINGBOOT STOP...."
- SPRING_PID=$(pgrep -f v1-0.0.1-SNAPSHOT.jar)
- kill -9 $SPRING_PID
- 권한 부여 후 실행 (서버 종료)
- 서버를 재시작하기 위한 스크립트 생성 vi sprint-restart.sh
- SPRING_PID=$(pgrep -f v1-0.0.1-SNAPSHOT.jar)
- SPRING_PATH="/home/ubuntu/aws-v1/build/libs/v1-0.0.1-SNAPSHOT.jar"
- echo $SPRING_PID
- echo $SPRING_PATH
- echo "스프링 종료된 상태...."
- echo "스프링 재시작 - $(date)" 1>>/home/ubuntu/cron_restart/spring-restart.log
- nohup java -jar $SPRING_PATH 1>log.out 2>err.out &
- else
-   echo "스프링 시작된 상태...."
- fi
- 권한 부여 후 실행하면 서버 백그라운드 실행 + spring-restart.log에 재시작 시간 기록
- vi deploy.sh
- /# 1. 배포 프로세스 (/은 제외)
- echo "deploy start... 기존 서버가 시작되어 있다면 종료를 하고 배포를 시작해야함"
- echo "1. JDK install - x"
- echo "2. github project download - o"
- echo "3. gradlew 실행 권한 주기 - o"
- echo "4. project build  - o"
- echo "5. ubuntu timezone setting - x"
- echo "6. nohup으로 springboot 실행시키기 - o"
- /# 2. 스프링서버 종료시 재시작 (/은 제외)
- echo "crontab 등록 - spring restart..."
- crontab -l >crontab_new
- echo "* * * * * /home/ubuntu/cron_restart/spring-restart.sh" 1>>crontab_new
- crontab crontab_new
- rm crontab_new
- 권한 부여 후 실행하여 서버가 꺼져있는 경우 1분마다 확인하여 spring-restart.sh로 재시

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

### 고민 5
- 재배포시 서버가 안 멈추게 할 수 없을까?
- 재배포시 ec2를 새로 생성해서 거기를 재배포를 하고 (apt update, JDK 설치, crontab 재등록) 성공적으로 재배포를 종료
