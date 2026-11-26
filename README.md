# K8s 실습 저장소 (VM -01 세트)

## 구성
| 경로 | 내용 |
|---|---|
| `app/index.html` | 애플리케이션 소스 (nginx가 보여주는 웹 페이지) |
| `k8s/deployment.yaml`, `k8s/service.yaml` | CI/CD 대상 앱 `cicd-web` 배포 매니페스트 (NodePort 30090) |
| `kustomization.yaml` | ArgoCD가 읽는 진입점. `app/index.html`로 ConfigMap 생성 |
| `k8s-basic/` | 1~3번 실습 매니페스트 (web, NodePort 30080, Istio 라우팅) — 기록용, ArgoCD 대상 아님 |

## 자동 배포 흐름 (과제 4.1)
1. GitHub에서 `app/index.html` 수정 → Commit
2. ArgoCD가 저장소 변경을 감지 → 새 ConfigMap 생성 → `cicd-web` Pod 자동 교체
3. bastion에서 `curl -s 10.255.27.160:30090 | grep -i title` 로 바뀐 내용 확인
