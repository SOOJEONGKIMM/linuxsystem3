# LinuxSystemProgramming
# Project 3

◯ ssu_crontab 프로그램 기본 사항
- 사용자가 주기적으로 실행하는 명령어를 “ssu_crontab_file”에 저장 및 삭제하는 프로그램
- 주기적으로 “ssu_crontab_file”에 저장된 명령어를 실행시킬 “ssu_crond” 디몬 프로그램

ü “ssu_crond”는 운영체제 시작 시 함께 실행되어 “ssu_crontab_file”에 저장된 명령어를 주기적
으로 실행시킴
- “ssu_crontab_file”에 저장된 명령어가 정상적으로 수행된 경우만 “ssu_crontab_log”로그파일에
다음 사항을 기록

ü 출력형태
- [수행시간] 프롬프트명령어 실행주기 명령어 명령어 옵션
- [수행시간] : 요일 월 날짜 명령어 실행시작 시간
- 프롬프트 명령어 : add, remove, run 중 하나표시
Ÿ add : ssu_crontab을 통해 명령어가 저장된 경우
Ÿ remove : ssu_crontab을 통해 명령어가 삭제된 경우
Ÿ run : ssu_crond를 통해 명령어가 수행된 경우
- 실행주기 : 분 시 일 월 요일
Ÿ 각 항목의 값으로 들어가는 내용은 설계 및 구현에서 설명
- 명령어와 명령어 옵션
ü “ssu_crontab_file”에 명령어가 추가, 제거 될 때마다 “ssu_crontab_log”로그파일에 기록
ü “ssu_crond”가 “ssu_crontab_file”에 저장된 명령어를 주기적으로 실행할 때마다
“ssu_crontab_log”로그파일에 기록

◯ ssu_rsync 프로그램 기본 사항
- 명령어 형태 : ssu_rsync src(소스파일 또는 디렉토리) dst(목적 디렉토리)
- 인자로 주어진 src 파일 혹은 디렉토리를 dst 디렉토리에 동기화
- 동기화 시 src 파일이 dst 디렉토리 내 동일한 파일(파일 이름과 파일 크기, 수정시간이 같은 경우)로
존재할 경우 해당 파일은 동기화 하지 않음
- 예1) src가 일반 파일인 경우 src와 동일한 이름의 dst 디렉토리 내에 일반파일로 존재할 경우 해당
파일은 동기화 하지 않음
- 예2) src가 디렉토리 파일인 경우 src 디렉토리 내 모든 파일과 dst 디렉토리 내 모든 파일을 비교하
여 동일한 일반 파일이 존재할 경우 해당 파일은 동기화하지 않음
- 동기화 작업 도중 SIGINT가 발생하면 동기화 작업이 취소되고 dst 디렉토리 내 파일들이 동기화하지
않은 상태로 유지되어야 함
- 단 동기화 중에는 사용자가 동기화 중인 파일을 open 하는 것은 허용하지 않음
- src 파일 dst 디렉토리 내 파일의 동기화가 모두 완료되면 “ssu_rsync_log”로그파일에 기록
ü 출력형태
- [동기화 완료시간] 명령어 및 명령어 옵션
파일 이름 파일 크기
- 동기화 완료시간 : 요일 월 날짜 명령어 동기화 완료시간
- 명령어 및 명령어 옵션
- 파일 이름 : 동기화한 파일의 이름(src 디렉토리 기준 상대경로)
- 파일 크기 : 동기화한 파일의 크기, 단. 파일이 삭제된 경우 delete를 기록

◯ 설계 및 구현
- 가) ssu_crontab
ü ssu_crontab 실행 후 다음과 같은 프롬프트 출력
- 프롬프트 모양 : “ssu_crontab_file”에 저장된 모든 명령어 출력 및 개행 후, 공백 없이 학번,
‘>’ 문자 출력.
- 프롬프트에서 실행 가능한 명령어 : add, remove, exit
- 이외 명령어 입력 시 에러 처리 후 프롬프트로 제어가 넘어감
- 엔터만 입력 시 프롬프트 재출력
- exit 명령이 입력될 때까지 위에서 지정한 실행 가능 명령어를 입력받아 실행

ü add <실행주기> <명령어>
- “ssu_crontab_file”에 실행주기와 명령어가 기록되어야 하며 “ssu_crontab_file”파일이 없을 경
우 생성함
- 실행주기
Ÿ 실행주기의 각 항목은 분 시 일 월 요일 의 5가지 항목으로 구성됨
Ÿ 각 항목은 분(0-59), 시(0-23), 일(1-31), 월(1-12), 요일(0-6, 0은 일요일 1~6순으로 월요일에서
토요일을 의미)의 범위를 가짐
Ÿ 각 항목의 값은 ‘*’, ‘-’, ‘,’, ‘/’ 기호를 사용하여 설정할 수 있음
§ ‘*’: 해당 필드의 모든 값을 의미함
§ ‘-’: ‘-’으로 연결된 값 사이의 모든 값을 의미함(범위 지정)
§ ‘,’: ‘,’로 연결된 값들 모두를 의미함(목록)
§ ‘/’: 앞에 나온 주기의 범위를 뒤에 나온 숫자만큼 건너뛰는 것을 의미함
Ÿ 예) 0 0,12 */2 * * -> 2일마다 0시 0분, 12시 0분에 작업수행
- 명령어 인자에는 주기적으로 수행할 명령어 및 해당 명령어의 옵션까지 모두 입력
- 실행주기의 입력이 잘못 된 경우 에러 처리 후 프롬프트로 제어가 넘어감
- add 프롬프트 명령어를 통해 명령어가 “ssu_crontab_file”파일에 저장되면 “ssu_crontab_log”로
그파일에 로그를 남김

ü remove <COMMAND_NUMBER>
- 옵션으로 입력한 번호의 명령어를 제거
- 잘못된 COMMAND_NUMBER가 입력된 경우 에러 처리 후 프롬프트로 제어가 넘어감
- COMMAND_NUMBER를 입력하지 않은 경우 에러 처리 후 프롬프트로 제어가 넘어감
- remove 프롬프트 명령어를 통해 명령어가 “ssu_crontab_file”파일에서 삭제되면
“ssu_crontab_log”로그파일에 로그를 남김

ü exit
- 프로그램 종료


- 나) ssu_crond
ü 운영체제 시작 시 함께 실행되어 “ssu_crontab_file”에 저장된 명령어를 주기적으로 실행시키는
디몬 프로그램
ü “ssu_crond”는 “ssu_crontab_file”을 읽어 주기적으로 명령어를 실행해야 함
ü “ssu_crontab_file”에 명령어가 저장 및 삭제되면 이를 반영 해야함
ü “ssu_crond”를 통해 명령어가 수행되면 “ssu_crontab_log”로그파일에 로그를 남김

- 다) ssu_rsync [option] <src> <dst>
ü “ssu_rsync”는 인자로 주어진 src 파일 혹은 디렉토리를 dst 디렉토리에 동기화함
ü 동기화 시 src 파일이 dst 디렉토리 내 동일한 파일(파일 이름과 파일 크기, 수정시간이 같은 경
우)이 존재하지 않는 경우 src 파일을 dst 디렉토리 내에 복사함
ü 동기화 시 dst 디렉토리 내에 파일 이름이 같은 다른 파일(파일 이름은 같으나 파일 크기 혹은
수정시간이 다른 경우)이 존재할 경우 dst 디렉토리 내의 파일을 src 파일로 대체함
ü dst 디렉토리 내의 동기화된 파일은 “ssu_rsync”프로그램에서 src 디렉토리의 파일과 같은 파일
로 인식할 수 있어야함
ü src 인자는 파일 및 디렉토리 모두 허용
- 상대경로와 절대경로 모두 입력 가능
- 인자로 입력받은 파일 혹은 디렉토리를 찾을 수 없으면 usage 출력 후 프로그램 종료
- 인자로 입력받은 파일 혹은 디렉토리의 접근권한이 없는 경우 usage 출력 후 프로그램 종료
ü dst 인자는 디렉토리만 허용
- 상대경로와 절대경로 모두 입력 가능
- 인자로 입력받은 디렉토리를 찾을 수 없으면 usage 출력 후 프로그램 종료
- 인자로 입력받은 디렉토리가 디렉토리 파일이 아니라면 usage 출력 후 프로그램 종료
- 인자로 입력받은 디렉토리의 접근권한이 없는 경우 usage 출력 후 프로그램 종료
ü ‘-r’옵션을 설정하지 않은 경우 인자로 지정한 src의 서브디렉토리는 동기화하지 않음
ü ‘-m’옵션을 설정하지 않은 경우 src 디렉토리에 존재하지 않는 파일 및 디렉토리가 dst 디렉토
리에 존재할 수 있음
ü 동기화 작업 도중 SIGINT가 발생하면 동기화 작업이 취소되고 dst 디렉토리 내 파일들이 동기화
하지 않은 상태로 유지되어야 함
  
- 라) -r
ü -r 옵션이 설정되면 src의 서브디렉토리 내의 파일 또한 동기화함
ü dst 디렉토리에 동기화할 서브디렉토리가 없는 경우 디렉토리를 생성해 주어야함
  
- 마) -t
ü -t 옵션이 설정되면 동기화가 필요한 대상들을 묶어 한번에 동기화 작업을 수행
ü tar를 활용하여 묶음
ü tar파일 전송 완료 후 묶음 해제하여 정확한 위치에 동기화
ü -t 옵션 사용 시 로그기록 방식이 달라짐
ü 로그기록 방식
- [동기화 완료시간] 명령어 및 명령어 옵션
totalSize 전송된 tar 파일의 크기
파일이름
- 동기화 완료시간 : 요일 월 날짜 명령어 동기화 완료시간
- 명령어 및 명령어 옵션
- totalSize : 로그 출력양식용 텍스트
- 전송된 tar 파일의 크기 : 동기화가 필요한 대상들을 묶은 tar 파일의 크기
- 파일 이름 : 동기화한 파일의 이름(src 디렉토리 기준 상대경로)
  
- 바) -m
ü -m 옵션이 설정되면 dst 디렉토리에 src 디렉토리에 없는 파일 및 디렉토리가 존재할 경우 dst
디렉토리에서 해당하는 파일 및 디렉토리를 삭제함
ü src 디렉토리에 없는 파일 및 디렉토리가 삭제되면 “ssu_rsync_log”로그파일에 로그를 남김
