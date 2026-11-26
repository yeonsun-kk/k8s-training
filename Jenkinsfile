// CI 파이프라인 (Jenkins @ bastion)
// GitHub 변경 감지(1분마다) → 소스 검사 → ArgoCD 배포 대기 → bastion에서 실제 응답 확인
pipeline {
  agent any
  triggers { pollSCM('* * * * *') }          // 1분마다 GitHub main 변경 확인
  options { timestamps() }
  environment {
    APP_URL = 'http://10.255.27.160:30090'    // cicd-web NodePort
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'git log -1 --oneline'
      }
    }
    stage('Test') {
      steps {
        sh '''
          set -e
          echo "[1] index.html 에 <title> 이 있는지"
          grep -q "<title>" app/index.html
          VERSION=$(sed -n 's:.*<title>\\(.*\\)</title>.*:\\1:p' app/index.html)
          echo "    → 배포할 버전: $VERSION"
          echo "$VERSION" > expected_title.txt
          echo "[2] 매니페스트 필수 파일이 있는지"
          test -f kustomization.yaml && test -f k8s/deployment.yaml && test -f k8s/service.yaml
          grep -q "app/index.html" kustomization.yaml
          echo "    → OK"
        '''
      }
    }
    stage('Deploy (ArgoCD)') {
      steps {
        echo 'ArgoCD가 GitHub main 변경을 감지해 클러스터에 자동 배포합니다 (최대 약 1~2분).'
      }
    }
    stage('Verify') {
      steps {
        sh '''
          EXPECTED=$(cat expected_title.txt)
          for i in $(seq 1 18); do
            ACTUAL=$(curl -s --max-time 5 "$APP_URL" | sed -n 's:.*<title>\\(.*\\)</title>.*:\\1:p')
            echo "$(date +%T) 현재 응답: $ACTUAL"
            if [ "$ACTUAL" = "$EXPECTED" ]; then
              echo "배포 확인 완료: $ACTUAL"
              exit 0
            fi
            sleep 10
          done
          echo "3분 안에 새 버전이 확인되지 않았습니다 (기대값: $EXPECTED)"
          exit 1
        '''
      }
    }
  }
}
