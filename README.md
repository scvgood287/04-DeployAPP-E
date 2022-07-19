# 04-DeployAPP-E

4주차 과제 / Amazon EC2를 이용하여, Free-Tier 서버를 생성합니다.

## 과제 분석

간단하게 AWS EC2에 배포하는 게 다이다.  
분석이라 할 것 까지는 없지만, 저번에 실패한 Github Actions Workflows 도 사용해 볼 겸 할 일들을 나열해 보았다.

- [x] AWS EC2 인스턴스 생성
- [x] AWS Elastic IP
- [x] Dockerfile
- [x] Github Access Token & Github Secrets
- [x] Github Actions Workflows
- [x] Github Runners
- [x] Nginx

## AWS EC2 인스턴스 생성

우선 AWS 의 리전 설정을 해둔다.  
아마 서울로 되어 있긴 하겠지만 혹시 모르니 확인.  
확인은 우측 상단 헤더에서 할 수 있다.

---

<img width="1205" alt="스크린샷 2022-07-19 오전 9 54 06" src="https://user-images.githubusercontent.com/79984416/179640989-bff6ee34-e241-424b-b9ff-4f6235cd1f23.png">

AWS Console 에서 EC2 에 들어가면 다음과 같은 화면이 나온다.  
우측 상단의 인스턴스 시작 클릭.

---

<img width="760" alt="스크린샷 2022-07-19 오전 9 56 01" src="https://user-images.githubusercontent.com/79984416/179641153-bdae76b5-6d4b-4344-9875-1845c6044607.png">

이름은 어차피 수정 가능하니 편한 대로 지어준다. 나의 경우엔 그냥 Pre-Onboarding-E/DeployAPP 으로 지어줬다.  
그다음도 그냥 Quick Start 의 Ubuntu 를 사용했다.  
까먹진 않을 테지만 추후 Github Runners 설정에 필요하니 기억해두자.

---

<img width="806" alt="스크린샷 2022-07-19 오전 10 00 07" src="https://user-images.githubusercontent.com/79984416/179641526-fee6319e-f076-43c9-8b1e-740a485aae87.png">

그 이외의 설정은 전부 기본 설정을 사용했는데, 인스턴스에 접속할 때 사용할 키 페어를 설정해두면 길~게 봤을 때 매우 편해질 테니 설정해두자.  
키 페어 생성은 이렇게 하면 된다. https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/EMRforDynamoDB.Tutorial.EC2KeyPair.html  
끝났다면 인스턴스 시작!

---

<img width="841" alt="스크린샷 2022-07-19 오전 10 03 34" src="https://user-images.githubusercontent.com/79984416/179641909-0f691f1c-bbc8-4ef1-bfac-b5696e0d8a79.png">
<img width="822" alt="스크린샷 2022-07-19 오전 10 08 03" src="https://user-images.githubusercontent.com/79984416/179642269-78fb942b-abb6-459e-af6a-988bee7d3360.png">

위와 같이 인스턴스 상태가 실행 중이고, 상태 검사가 통과되었다면 생성 완료!  
그런데 이대로면 접속이 안 된다...  
이유는 보안의 인바운드 규칙을 설정하지 않았기 때문이다.

---

<img width="1418" alt="스크린샷 2022-07-19 오전 11 56 25" src="https://user-images.githubusercontent.com/79984416/179654581-89909fe1-be69-433e-bd70-aa9d1f4d7109.png">
<img width="1389" alt="스크린샷 2022-07-19 오전 11 58 13" src="https://user-images.githubusercontent.com/79984416/179654802-a1d4b868-5c88-4749-9595-6734d9564c76.png">

사진과 같이 인스턴스를 클릭한 후, 보안 탭의 보안 그룹에 파란 글씨로 링크가 걸려있는 부분을 클릭해 보자.  
그럼 두 번째 사진과 같은 화면이 나오는데 우측 하단의 인바운드 규칙 편집을 클릭하자.

---

<img width="1565" alt="스크린샷 2022-07-19 오후 12 00 20" src="https://user-images.githubusercontent.com/79984416/179655055-0b5ee0f7-834b-4bcd-83e1-f108c6fdd6b6.png">

이번엔 그냥 간단한 접속만 해보려 하니 사진과 같이 설정해 준 후 규칙 저장하면 끝이다!  
실제론 더 할게 많을 테지만 우선 이 정도만...  
이제 Elastic IP 를 할당하여 연결해 보자.

## AWS Elastic IP

다른 말로 탄력적 IP 라고도 한다.  
EC2 인스턴스를 생성하여 서버를 실행시키면 고정 IP 가 아닌 동적 IP 를 할당받는다.  
그렇기에 인스턴스를 중지시키고 다시 실행시키면 IP 가 변경되는 문제가 발생하므로,  
이용해 IP 를 고정시키기 위해 탄력적 IP 를 사용하는 것이다.

<img width="887" alt="스크린샷 2022-07-19 오전 10 23 53" src="https://user-images.githubusercontent.com/79984416/179643838-69151f62-d062-4b27-8db7-55fbc4a5f6fc.png">

https://aws.amazon.com/ko/premiumsupport/knowledge-center/elastic-ip-charges/  
그렇다고 하니 탄력적 IP 를 할당받으면 재빨리 연결해 주고, 사용하지 않으면 릴리스 해주자...

---

<img width="217" alt="스크린샷 2022-07-19 오전 10 12 08" src="https://user-images.githubusercontent.com/79984416/179642600-8ad81b5b-2861-4b73-a070-a18670d7c4f6.png">

인스턴스 생성 후 왼편의 목록 중 탄력적 IP 을 클릭 후 우측 상단의 탄력적 IP 주소 할당 클릭.  
딱히 이번엔 상세 설정하지 않았으니 그대로 할당해 준다.

---

<img width="1433" alt="스크린샷 2022-07-19 오전 10 18 15" src="https://user-images.githubusercontent.com/79984416/179643244-3d5d29f1-8c22-4d77-a80a-60c6e628da29.png">

그럼 다음과 같이 할당되는데, 여기서 할당된 IPv4 주소 클릭 후 탄력적 IP 주소 연결을 클릭.

---

<img width="842" alt="스크린샷 2022-07-19 오전 10 20 08" src="https://user-images.githubusercontent.com/79984416/179643438-86043509-704b-4d3c-aec4-7681eac70b66.png">

밑의 인스턴스 선택에서 방금 만든 인스턴스를 선택해 연결해 주면 끝이다!

## Dockerfile

<img width="250" alt="스크린샷 2022-07-19 오전 10 26 59" src="https://user-images.githubusercontent.com/79984416/179644173-dfab3845-4cae-414e-bc4c-9dd80055b843.png">

이번 과제용으로 생성한 NestJS 프로젝트이다.  
과제에서 IP/api/hello 로 접속하였을 때 Hello 가 나와야 한다 하니 그 부분만 살짝 건드려줬다.

```
nest new <project-name>
```

이 중 Docker Image 를 빌드 하기 위해 루트 경로에 Dockerfile 을 생성해 아래 내용을 써넣었다.  
여러 가지 할 수 있겠지만 이번엔 진짜 간단하게만 작성했다.

```
FROM node:16-alpine3.11

WORKDIR /app
COPY . .
RUN yarn install
RUN yarn build
CMD ["yarn", "start:prod"]
```

.dockerignore 도 설정해주자

```
node_modules
dist
```

## Github Access Token & Github Secrets

Github Actions Workflows 를 작성하기 전, GHCR(GitHub Container Registry) 로그인에 필요한 Github Secrets 부터 설정해 주자.  
GHCR 에 대한 설명은 여기에...  
https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry

<img width="1109" alt="스크린샷 2022-07-19 오전 11 11 08" src="https://user-images.githubusercontent.com/79984416/179649094-f1d7d24a-8ba0-439b-8d16-170c153d656f.png">
우선 Github 계정 Settings 에서 Developer settings 에 들어가준다.  
여기서 Personal access tokens 에 들어가 Generate new token 을 클릭.

---

<img width="791" alt="스크린샷 2022-07-19 오전 11 13 11" src="https://user-images.githubusercontent.com/79984416/179649338-66270e37-8f45-4287-a23a-bb2781249dd9.png">

사진과 같이 이름과 만료 기간을 설정해 주고 밑에 네 가지를 체크해주자.  
나는 만료 기간 없음, 이름은 GHCR_TOKEN 으로 지정했다.  
추후 Secrets 설정 때 사용할 테니 생성한 토큰은 따로 복사해두자.

---

<img width="1329" alt="스크린샷 2022-07-19 오전 11 16 10" src="https://user-images.githubusercontent.com/79984416/179649666-2b1717e0-1947-495f-be82-0e8369ca7de4.png">  
<img width="818" alt="스크린샷 2022-07-19 오전 11 18 45" src="https://user-images.githubusercontent.com/79984416/179649955-50ad9702-8b56-421b-b791-33941134e16c.png">

다음과 같이 적용시킬 Repository 의 Settings - Secrets - Actions 에서 New Repository secret 을 클릭.  
이름과 값을 넣고 Add secret 클릭. 나는 이름은 똑같이 설정해 줬다.

## Github Actions Workflows

이번에 꼭 해보고 싶었던 Github Actions Workflows 를 이용한 CI/CD 파트이다...  
루트 경로에 /.github/workflows/<파일명>.yaml 으로 파일을 생성해 준다.  
.github/workflows 폴더에 넣어놓으면 Github 에서 파일을 자동으로 읽어서 등록해 준다.

```yaml
name: CI/CD

# main branch 에 push 될 때 동작시키겠단 뜻이다.
# 참고로 그냥 push 뿐만 아니라 merge 도 push event 를 발생하니
# push 나 merge 가 발생하면 동작한다.
on:
  push:
    branches: [main]

# 사용할 환경 변수를 미리 지정한다.
env:
  DOCKER_IMAGE: ghcr.io/${{ github.actor }}/trading
  VERSION: ${{ github.sha }}
  NAME: nestjs_cicd

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      # 소스 코드를 컨테이너 안으로 Checkout
      - name: Checkout source code
        uses: actions/checkout@v2
      # 가상의 컨테이너 안에 Docker 가 돌아갈 수 있는 환경 설치
      - name: Set up docker buildx
        id: buildx
        uses: docker/setup-buildx-action@v1
      - name: Cache docker layers
        uses: actions/cache@v2
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ env.VERSION }}
          restore-keys: |
            ${{ runner.os }}-buildx-
      # Github Secrets 에 설정해둔 GHCR_TOKEN 으로 GHCR(GitHub Container Registry) 로그인
      - name: Login to ghcr
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GHCR_TOKEN }}
      # GHCR 에 Docker Image 를 빌드 한 후 Push
      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          # 여기서 steps.buildx 는 두 번째 단계에서 id 를 buildx 로 설정해서 불러온 것이다.
          builder: ${{ steps.buildx.outputs.name }}
          push: true
          tags: ${{ env.DOCKER_IMAGE }}:latest

  deploy:
    needs: build
    name: Deploy
    runs-on: [self-hosted, label-nestjs]
    steps:
      # GHCR 에 로그인
      - name: Login to ghcr
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GHCR_TOKEN }}
      # 실행 중인 Docker Container 중지, 이전 버전 Container 와 Image 삭제, 새로운 Image 로 Container 실행.
      - name: Docker run
        # 외부에서 8080 포트로 요청이 들어오면 내부의 5000 포트로 요청을 넘겨준다.
        run: |
          docker stop ${{ env.NAME }} && docker rm ${{ env.NAME }} && docker rmi ${{ env.DOCKER_IMAGE }}:latest
          docker run -d -p 8080:5000 --name nestjs_cicd --restart always ${{ env.DOCKER_IMAGE }}:latest
```

Workflow 는 job 들로 구성이 되어있는데, 나는 build 와 deploy 로 구성했다.  
그리고 job 은 step 으로 나뉘며, step 은 name 을 기준으로 실행된다.  
자세한 내용은 주석을 달아놨다.

runs-on 이 중요한 부분인데, 꼭 self-hosted 를 적어놔야 한다.  
그래야 EC2 에 설정할 Github Runners 가 실행된다.  
뒤의 label 은 Runner 를 구분하기 위한 것이니 편하게 지정해서 넣어주자.

## Github Runners

<img width="1315" alt="스크린샷 2022-07-19 오전 11 26 46" src="https://user-images.githubusercontent.com/79984416/179650897-b4800436-8ac2-4f3b-9701-83ee92aceaa2.png">

Repository - Settings - Actions - Runners 에서 New self-hosted runner 클릭!

---

<img width="781" alt="스크린샷 2022-07-19 오전 11 35 23" src="https://user-images.githubusercontent.com/79984416/179651972-dde204c6-8d26-47ca-b6f4-b6614c908bd9.png">

맨 처음 EC2 를 생성할 때, 우린 ubuntu x64 로 생성했으니, Linux x64 를 선택.  
혹시 다른 OS 로 인스턴스를 생성했을 땐 그에 맞게 설정해 주자.  
그리고 EC2 인스턴스에 접속해 주는데, EC2 생성 시 키 페어를 설정했다면 ssh 로 자신의 터미널로 접속해도 좋고,  
그냥 EC2 콘솔에서 직접 접속해도 좋다.  
그 후엔 간단하다! 그저 따라 치기만 하면 된다.

---

<img width="837" alt="스크린샷 2022-07-19 오전 11 38 30" src="https://user-images.githubusercontent.com/79984416/179652311-5f53d413-2487-44b2-a8a9-38897bc41570.png">

```
./config.sh --url https://github.com/scvgood287/03-BossRaid-E --token <TOKEN>
```

따라 치다가 이 부분을 입력하면 위 사진과 같은 화면이 나올 텐데,  
첫 번째는 그냥 엔터,  
두 번째는 복수의 Runner 를 등록할 때 Github 상에서 구분할 Runner 의 이름을 지정하면 되고,  
세 번째는 실행될 Runner 를 구분하기 위한 것이다.

두 번째는 .github/workflows/<파일명>.yaml 의 env 의 DOCKER_IMAGE 의 뒷부분과 맞춰 trading 으로 했다. (의미는 없음)  
세 번째는 Deploy 의 runs-on 에서 지정한 라벨인 label-nestjs 로 지정했다.  
둘 다 임의로 변경 가능하다!

후에 Repository - Settings - Actions - Runners 에서 확인 가능하다.

---

<img width="390" alt="스크린샷 2022-07-19 오전 11 47 52" src="https://user-images.githubusercontent.com/79984416/179653512-f22509b8-422b-4f95-aabf-e274c0771a09.png">

마지막 ./run.sh 를 입력했을 때 위 사진과 같이 나온다면 성공이다!  
./run.sh 은 다른 작업을 하지 못하므로 nohup ./run.sh & 를 이용해 백그라운드로 Runner 를 실행시킨다.  
이제 main branch 에 push 나 merge 가 생기면 자동으로 이미지가 빌드 되고 EC2 에 배포된다!

## Nginx

마지막 Nginx 이다. Docker Container 내부에 Nginx 를 설정할까 EC2 인스턴스에 직접 설정할까 고민하다가  
EC2 인스턴스에 직접 설정하기로 결정했다.  
아래의 명령어를 입력하여 Nginx 를 설치하고 설정해 보자!

```
sudo apt update
sudo apt install nginx
cd /etc/nginx/conf.d && sudo touch custom.conf && sudo vim custom.conf
```

마지막 줄에서 /etc/nginx/conf.d 라는 폴더에 custom.conf 라는 파일을 만들어 편집하는데,  
/etc/nginx/nginx.conf 라는 파일을 열어보면,  
conf.d 라는 폴더 안의 .conf 파일들을 모두 불러오고 있기 때문에 이런 식으로 했다.  
custom.conf 에서 아래와 같이 입력해 보자.

```
server {
  listen 80;

  location / {
    proxy_pass http://localhost:8080;
  }
}
```

esc 를 누르고 :wq! 입력 후 빠져나오자. (저장하고 나가기)  
간단하게 설명하자면, 80번 포트로 들어온 요청을 8080 포트로 넘겨준다는 의미다.  
왜 8080 포트인지는 위의 workflow 설정의 마지막 줄을 보면 알 수 있다.

```yaml
# .github/workflows/cicd.yaml

...

- name: Docker run
        # 외부에서 8080 포트로 요청이 들어오면 내부의 5000 포트로 요청을 넘겨준다.
        run: |
          docker stop ${{ env.NAME }} && docker rm ${{ env.NAME }} && docker rmi ${{ env.DOCKER_IMAGE }}:latest
          docker run -d -p 8080:5000 --name nestjs_cicd --restart always ${{ env.DOCKER_IMAGE }}:latest
```

```ts
// main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { cors: true });

  app.setGlobalPrefix('/api');

  // 여기!
  await app.listen(5000);
}
bootstrap();
```

주석의 설명과 같이 컨테이너를 8080 포트로 실행했기 때문에, 80번 포트로 들어온 요청을 컨테이너의 포트인 8080 포트로 넘겨주고,  
다시 컨테이너가 내부의 5000 포트로 넘겨준다.  
대충 다 80으로 통일해도 괜찮지만 나중에 헷갈릴까 봐 지금 미리 집고 넘어가려 해봤다...  
설정이 끝나면 nginx -s reload 로 Nginx 를 재시작해주자.

이걸로 끝이다! 라고 하고 싶지만...

## 몇 가지 오류들

이번 배포에서 겪은 몇 가지 오류들을 적어두려 한다.

### Docker

<img width="1646" alt="스크린샷 2022-07-19 오후 12 32 11" src="https://user-images.githubusercontent.com/79984416/179658664-b22c42ad-1912-4281-ae38-a31e6abcda3f.png">

```
Error: Unable to locate executable file: docker. Please verify either the file path exists or the file can be found within a directory specified by the PATH environment variable. Also check the file mode to verify the file is executable.
```

뜬금없이 docker...?  
workflow 의 yaml 을 잘 살펴보니... Deploy 부분부턴 runs-on 이 self-hosted 였던 걸 깨달았다.  
아, EC2 에 docker 설치를 깜빡했네. 싶어서 바로 설치  
https://insight.infograb.net/docs/aws/installing-docker-on-aws-ec2/  
위 링크를 참고하여 설치 후 sudo 없이 docker 명령어 사용 가능하게 설정해두었다.

### 권한

<img width="1234" alt="스크린샷 2022-07-19 오후 12 44 58" src="https://user-images.githubusercontent.com/79984416/179660068-1c7350b5-6128-4adf-94f7-6edf8a70545e.png">

```
Error: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/auth": dial unix /var/run/docker.sock: connect: permission denied
```

/var/run/docker.sock: connect: permission denied 라...  
무식하게 /var/run/docker.sock 에 권한을 줘보자 싶어 EC2 에 접속해

```
sudo chmod 777 /var/run/docker.sock
```

권한 부여를 해주니 바로 해결되었다!  
이제 진짜 끝!!!

## 아쉬운 점

Docker Compose 를 활용해 여러 개의 컨테이너를 올릴 때도 해보고 싶다.  
CI/CD 때 Test 까지 진행하여 통과해야 배포되는 식으로 진행해 보고 싶다.  
따로 설정을 안 해서 URL 이 13.209.27.248 이런 모양이다. 나중에 Route 53 을 써서 이쁘게 꾸며주고 싶다.

## 결과

<img width="461" alt="스크린샷 2022-07-19 오후 1 45 15" src="https://user-images.githubusercontent.com/79984416/179666624-030d176d-ebc9-43eb-9b44-887e43ccb4c5.png">
성공!

## 참고

https://velog.io/@soosungp33/Github-Actions%EC%9C%BC%EB%A1%9C-AWS-EC2%EC%97%90-CICD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0

https://insight.infograb.net/docs/aws/installing-docker-on-aws-ec2/

https://stackoverflow.com/questions/69739714/got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket-at-uni

https://pstudio411.tistory.com/entry/EC2-%EB%B3%B4%EC%95%88-%EA%B7%B8%EB%A3%B9-%EC%84%A4%EC%A0%95

https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/EMRforDynamoDB.Tutorial.EC2KeyPair.html

https://aws.amazon.com/ko/premiumsupport/knowledge-center/elastic-ip-charges/

https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
