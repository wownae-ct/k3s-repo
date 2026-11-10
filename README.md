# k3s-repo

cortex-agent용 k3s 클러스터의 GitOps 매니페스트 저장소입니다. ArgoCD가 이 저장소를 보고 클러스터 상태를 맞춥니다.

## 동작 방식
- `root-app.yaml`이 `apps/`를 가리키는 App of Apps입니다. `apps/`에 Application을 추가하면 ArgoCD가 하위 앱을 만들고, 자동 동기화(prune·selfHeal)가 켜져 있습니다.
- 현재 연결된 앱은 `apps/study-agent.yaml`(→ `manifests/study-agent`: Deployment + Service)입니다. 이미지 태그는 CI가 이 저장소에 커밋해 갱신합니다.
- 커스텀 Jenkins 이미지를 빌드하는 Kaniko 파드는 `role=cicd` 노드에서 실행합니다.

## 구조
```
root-app.yaml            # App of Apps 진입점
apps/                    # ArgoCD Application 정의 (study-agent)
manifests/
  study-agent/           # Deployment, Service, GHCR pull secret(sealed)
  note-assistant/        # 준비 중
argocd/                  # ArgoCD 설치와 서버 설정, 저장소 접속 정보(sealed)
jenkins/
  jenkins-*.yaml         # StatefulSet, Service, RBAC(Kaniko·Secret), values
  custom-image/          # 플러그인을 이미지에 미리 넣은 Jenkins 이미지(Dockerfile, plugins.txt, Jenkinsfile)
```

## 커스텀 Jenkins 이미지
`jenkins/custom-image/Jenkinsfile`이 Kaniko 파드에서 이미지를 빌드해 GHCR에 올립니다(Docker 데몬 없이 클러스터 안에서 빌드). 재부팅 뒤 플러그인 설치 때문에 파드가 불안정해지는 것을 막으려고 플러그인을 이미지에 미리 넣었습니다.

## 시크릿
Sealed Secrets로 암호화한 파일(`*-sealedsecret.yaml`)만 커밋합니다. 평문 시크릿(`*-secret.yaml`)은 `.gitignore`로 막아 두었습니다.

## 상태
`apps/`와 `manifests/note-assistant`는 아직 준비 중인 항목이 있습니다.
