# Tasks: Phase IV - Local Kubernetes Deployment with AI-Generated Infrastructure

**Input**: Design documents from `/specs/004-phase4-kubernetes/`

**Prerequisites**:
- `specs/004-phase4-kubernetes/plan.md`
- `.specify/phase-4-kubernetes.md` (acts as the feature specification for Phase IV)

**Tests**: No automated tests required for this phase. Validation is done via `docker`, `kubectl`, `minikube`, `helm`, and manual verification.

**Organization**: Phase IV does not include explicit user stories in a `specs/004-phase4-kubernetes/spec.md` file. To keep tasks independently executable and incrementally deliverable, the work is organized into **derived user stories** (US1..US8) aligned to the plan/spec deliverables.

## Format: `- [ ] T### [P?] [US#] Description with file path`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[US#]**: Derived user story grouping for Phase IV
- Every task includes either (a) a concrete repo file path to create/modify, or (b) an explicit log/output capture file path under `specs/004-phase4-kubernetes/`.

## Path Conventions

- **Dockerfiles**: `frontend/Dockerfile`, `backend/Dockerfile`
- **Kubernetes manifests**: `k8s/*.yaml`
- **Helm chart**: `helm/todo-app/`
- **PHRs**: `history/prompts/004-phase4-kubernetes/`
- **Docs/Runbooks/Reports**: `specs/004-phase4-kubernetes/*.md`

---

## Phase 1: User Story 1 - Local Kubernetes environment ready (Priority: P1) 🎯 MVP

**Goal**: Minikube + kubectl + Helm are installed and a local cluster is running with ingress enabled.

**Independent Test**:
- `minikube status`
- `kubectl cluster-info && kubectl get nodes`
- `kubectl get pods -n ingress-nginx` (or the ingress addon namespace used by your Minikube)

- [ ] T001 [US1] Install Minikube and record `minikube version` output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T002 [P] [US1] Install kubectl and record `kubectl version --client` output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T003 [P] [US1] Install Helm 3 and record `helm version` output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T004 [US1] Start Minikube cluster (resources per plan) and record `minikube start ...` command used in specs/004-phase4-kubernetes/setup-report.md
- [ ] T005 [P] [US1] Enable NGINX Ingress addon and record output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T006 [P] [US1] Enable metrics-server addon and record output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T007 [US1] Verify cluster is running (`kubectl cluster-info && kubectl get nodes`) and record output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T008 [US1] Verify ingress controller pods are running (`kubectl get pods -n ingress-nginx`) and record output in specs/004-phase4-kubernetes/setup-report.md
- [ ] T009 [US1] Create repo directories for Phase IV artifacts: k8s/ and helm/todo-app/templates/ and specs/004-phase4-kubernetes/adr/

**Checkpoint**: Environment ready and repo has Phase IV directories.

---

## Phase 2: User Story 2 - AI-generated Dockerfiles for frontend/backend (Priority: P1) 🎯 MVP

**Goal**: Production-ready Dockerfiles exist for frontend and backend, are AI-generated, build successfully, and pass basic container runtime checks.

**Independent Test**:
- `docker build -t todo-frontend:latest ./frontend`
- `docker build -t todo-backend:latest ./backend`

### Frontend Dockerfile generation

- [ ] T010 [US2] Create frontend/.dockerignore (node_modules, .next, .git, .env*)
- [ ] T011 [US2] AI-generate frontend Dockerfile and save to frontend/Dockerfile (prompt and response must be captured in a PHR)
- [ ] T012 [US2] Record PHR for frontend Dockerfile generation in history/prompts/004-phase4-kubernetes/
- [ ] T013 [US2] Validate frontend Dockerfile builds successfully (docker build -t todo-frontend:latest ./frontend) and record build output in specs/004-phase4-kubernetes/docker-build-report.md
- [ ] T014 [US2] Test frontend container locally (docker run -p 3000:3000 todo-frontend:latest) and record result in specs/004-phase4-kubernetes/docker-run-report.md
- [ ] T015 [US2] Scan frontend image for vulnerabilities (trivy image todo-frontend:latest) and record summary in specs/004-phase4-kubernetes/security-scan-report.md

### Backend Dockerfile generation

- [ ] T016 [P] [US2] Create backend/.dockerignore (.venv, __pycache__, *.pyc, .git, .env*, tests/)
- [ ] T017 [P] [US2] AI-generate backend Dockerfile and save to backend/Dockerfile (prompt and response must be captured in a PHR)
- [ ] T018 [P] [US2] Record PHR for backend Dockerfile generation in history/prompts/004-phase4-kubernetes/
- [ ] T019 [US2] Validate backend Dockerfile builds successfully (docker build -t todo-backend:latest ./backend) and record build output in specs/004-phase4-kubernetes/docker-build-report.md
- [ ] T020 [US2] Test backend container locally (docker run -p 8000:8000 -e DATABASE_URL=... todo-backend:latest) and record result in specs/004-phase4-kubernetes/docker-run-report.md
- [ ] T021 [US2] Scan backend image for vulnerabilities (trivy image todo-backend:latest) and record summary in specs/004-phase4-kubernetes/security-scan-report.md

**Checkpoint**: Dockerfiles exist, images build and run locally.

---

## Phase 3: User Story 3 - Images built and loaded into Minikube (Priority: P1) 🎯 MVP

**Goal**: `todo-frontend:latest` and `todo-backend:latest` are available to the Minikube cluster.

**Independent Test**:
- `minikube image ls | grep todo`

- [ ] T022 [US3] Build frontend Docker image (docker build -t todo-frontend:latest ./frontend) and record image size in specs/004-phase4-kubernetes/image-report.md
- [ ] T023 [P] [US3] Build backend Docker image (docker build -t todo-backend:latest ./backend) and record image size in specs/004-phase4-kubernetes/image-report.md
- [ ] T024 [US3] Verify image sizes against plan targets and record in specs/004-phase4-kubernetes/image-report.md
- [ ] T025 [US3] Load frontend image into Minikube (minikube image load todo-frontend:latest) and record output in specs/004-phase4-kubernetes/minikube-image-report.md
- [ ] T026 [P] [US3] Load backend image into Minikube (minikube image load todo-backend:latest) and record output in specs/004-phase4-kubernetes/minikube-image-report.md
- [ ] T027 [US3] Verify images loaded in Minikube (minikube image ls | grep todo) and record output in specs/004-phase4-kubernetes/minikube-image-report.md

**Checkpoint**: Minikube can pull the local images (no ImagePullBackOff).

---

## Phase 4: User Story 4 - AI-generated raw Kubernetes manifests (Priority: P1) 🎯 MVP

**Goal**: Raw Kubernetes manifests exist under `k8s/` (AI-generated), validate with `kubectl apply --dry-run=client`, and are ready to deploy.

**Independent Test**:
- `kubectl apply --dry-run=client -f k8s/`

- [ ] T028 [P] [US4] AI-generate k8s/namespace.yaml
- [ ] T029 [P] [US4] AI-generate k8s/configmap.yaml
- [ ] T030 [P] [US4] AI-generate k8s/secret.yaml (template with base64 placeholders; do not commit real secrets)
- [ ] T031 [P] [US4] Record PHRs for namespace/configmap/secret generation in history/prompts/004-phase4-kubernetes/
- [ ] T032 [P] [US4] AI-generate k8s/deployment-frontend.yaml
- [ ] T033 [P] [US4] AI-generate k8s/service-frontend.yaml
- [ ] T034 [P] [US4] Record PHRs for frontend deployment/service generation in history/prompts/004-phase4-kubernetes/
- [ ] T035 [P] [US4] AI-generate k8s/deployment-backend.yaml
- [ ] T036 [P] [US4] AI-generate k8s/service-backend.yaml
- [ ] T037 [P] [US4] Record PHRs for backend deployment/service generation in history/prompts/004-phase4-kubernetes/
- [ ] T038 [US4] AI-generate k8s/ingress.yaml
- [ ] T039 [US4] Record PHR for ingress generation in history/prompts/004-phase4-kubernetes/
- [ ] T040 [US4] Validate all manifests with kubectl dry-run (kubectl apply --dry-run=client -f k8s/) and record output in specs/004-phase4-kubernetes/kubectl-validate-report.md
- [ ] T041 [US4] Fix any YAML validation errors (via AI re-prompting) and record what changed in specs/004-phase4-kubernetes/kubectl-validate-report.md
- [ ] T042 [US4] Commit k8s/ manifests (excluding real secret values) after validation

**Checkpoint**: `k8s/` validates cleanly and is ready to apply.

---

## Phase 5: User Story 5 - Deploy raw manifests and verify app access (Priority: P1) 🎯 MVP

**Goal**: App is deployed using raw manifests and is reachable via local ingress.

**Independent Test**:
- `kubectl get pods -n todo-app`
- `kubectl get ingress -n todo-app`
- `curl -I http://todo.local`
- `curl http://api.todo.local/health`

### Secret preparation (do not commit secrets)

- [ ] T043 [US5] Encode DATABASE_URL to base64 and record the command used (not the value) in specs/004-phase4-kubernetes/secrets-runbook.md
- [ ] T044 [US5] Encode OPENAI_API_KEY to base64 and record the command used (not the value) in specs/004-phase4-kubernetes/secrets-runbook.md
- [ ] T045 [US5] Generate and encode JWT_SECRET to base64 and record the command used (not the value) in specs/004-phase4-kubernetes/secrets-runbook.md
- [ ] T046 [US5] Update k8s/secret.yaml locally with base64 values (ensure real secrets are not committed)

### Deploy

- [ ] T047 [US5] Apply k8s/namespace.yaml
- [ ] T048 [US5] Apply k8s/configmap.yaml
- [ ] T049 [US5] Apply k8s/secret.yaml (local only; do not commit real values)
- [ ] T050 [US5] Apply k8s/deployment-frontend.yaml
- [ ] T051 [P] [US5] Apply k8s/deployment-backend.yaml
- [ ] T052 [US5] Apply k8s/service-frontend.yaml
- [ ] T053 [P] [US5] Apply k8s/service-backend.yaml
- [ ] T054 [US5] Apply k8s/ingress.yaml

### Verify

- [ ] T055 [US5] Wait for frontend rollout (kubectl rollout status deployment/todo-frontend -n todo-app)
- [ ] T056 [US5] Wait for backend rollout (kubectl rollout status deployment/todo-backend -n todo-app)
- [ ] T057 [US5] Verify pods running (kubectl get pods -n todo-app) and record output in specs/004-phase4-kubernetes/deploy-report.md
- [ ] T058 [US5] Verify services created (kubectl get svc -n todo-app) and record output in specs/004-phase4-kubernetes/deploy-report.md
- [ ] T059 [US5] Verify ingress created (kubectl get ingress -n todo-app) and record output in specs/004-phase4-kubernetes/deploy-report.md
- [ ] T060 [US5] Check pod logs for errors and record findings in specs/004-phase4-kubernetes/deploy-report.md
- [ ] T061 [US5] Verify liveness/readiness probes passing and record in specs/004-phase4-kubernetes/deploy-report.md

### Ingress and local DNS

- [ ] T062 [US5] Get Minikube IP (minikube ip) and record in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T063 [US5] Add /etc/hosts entries for todo.local and api.todo.local and record the exact line added in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T064 [US5] Verify ingress routing configuration and record in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T065 [US5] Test frontend HTTP access (curl -I http://todo.local) and record in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T066 [US5] Test backend health endpoint (curl http://api.todo.local/health) and record in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T067 [US5] Validate frontend in browser (http://todo.local) and record notes in specs/004-phase4-kubernetes/ingress-access-report.md
- [ ] T068 [US5] Verify frontend can reach backend (browser network/console) and record in specs/004-phase4-kubernetes/ingress-access-report.md

### Application validation (Phase III features)

- [ ] T069 [US5] Validate user registration works in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T070 [US5] Validate user login works in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T071 [US5] Validate authenticated API calls work in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T072 [US5] Validate logout works in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T073 [US5] Validate task creation in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T074 [US5] Validate task listing in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T075 [US5] Validate task update in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T076 [US5] Validate task completion toggle in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T077 [US5] Validate task deletion in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T078 [US5] Validate AI chat UI access (/chat) in Kubernetes and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T079 [US5] Validate AI-driven task creation via chat and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T080 [US5] Validate AI task listing via chat and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T081 [US5] Validate AI task update via chat and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T082 [US5] Validate AI task completion via chat and record in specs/004-phase4-kubernetes/app-validation-report.md
- [ ] T083 [US5] Validate AI chat responses are contextual and record in specs/004-phase4-kubernetes/app-validation-report.md

### Persistence and resilience

- [ ] T084 [US5] Verify tasks persist across pod restarts and record in specs/004-phase4-kubernetes/persistence-report.md
- [ ] T085 [US5] Verify users persist across pod restarts and record in specs/004-phase4-kubernetes/persistence-report.md
- [ ] T086 [US5] Verify chat history persists and record in specs/004-phase4-kubernetes/persistence-report.md

### Scaling and performance smoke checks

- [ ] T087 [US5] Scale backend up (kubectl scale deployment/todo-backend --replicas=3 -n todo-app) and record in specs/004-phase4-kubernetes/scale-report.md
- [ ] T088 [US5] Verify scaled pods running and record in specs/004-phase4-kubernetes/scale-report.md
- [ ] T089 [US5] Verify load balancing across pods (via logs) and record in specs/004-phase4-kubernetes/scale-report.md
- [ ] T090 [US5] Scale backend down (kubectl scale deployment/todo-backend --replicas=2 -n todo-app) and record in specs/004-phase4-kubernetes/scale-report.md
- [ ] T091 [US5] Measure frontend response time (curl -w "%{time_total}" -o /dev/null -s http://todo.local) and record in specs/004-phase4-kubernetes/perf-report.md
- [ ] T092 [US5] Measure backend health response time (curl -w "%{time_total}" -o /dev/null -s http://api.todo.local/health) and record in specs/004-phase4-kubernetes/perf-report.md
- [ ] T093 [US5] Test pod restart resilience and record in specs/004-phase4-kubernetes/resilience-report.md
- [ ] T094 [US5] Verify readiness probes gate traffic correctly and record in specs/004-phase4-kubernetes/resilience-report.md

**Checkpoint**: Raw-manifest deployment works end-to-end.

---

## Phase 6: User Story 6 - AI-generated Helm chart and Helm-based deployment (Priority: P2)

**Goal**: Helm chart exists, renders valid YAML, lints cleanly, and deploys the app.

**Independent Test**:
- `helm lint ./helm/todo-app`
- `helm template todo-app ./helm/todo-app | kubectl apply --dry-run=client -f -`

### Chart metadata and values

- [ ] T095 [US6] AI-generate helm/todo-app/Chart.yaml
- [ ] T096 [US6] Record PHR for Chart.yaml generation in history/prompts/004-phase4-kubernetes/
- [ ] T097 [US6] AI-generate helm/todo-app/values.yaml
- [ ] T098 [US6] Record PHR for values.yaml generation in history/prompts/004-phase4-kubernetes/

### Templates

- [ ] T099 [P] [US6] AI-generate helm/todo-app/templates/namespace.yaml
- [ ] T100 [P] [US6] AI-generate helm/todo-app/templates/configmap.yaml
- [ ] T101 [P] [US6] AI-generate helm/todo-app/templates/secret.yaml
- [ ] T102 [P] [US6] AI-generate helm/todo-app/templates/deployment-frontend.yaml
- [ ] T103 [P] [US6] AI-generate helm/todo-app/templates/deployment-backend.yaml
- [ ] T104 [P] [US6] AI-generate helm/todo-app/templates/service-frontend.yaml
- [ ] T105 [P] [US6] AI-generate helm/todo-app/templates/service-backend.yaml
- [ ] T106 [P] [US6] AI-generate helm/todo-app/templates/ingress.yaml
- [ ] T107 [US6] Record PHRs for Helm template generation in history/prompts/004-phase4-kubernetes/

### Helpers and notes

- [ ] T108 [US6] AI-generate helm/todo-app/templates/_helpers.tpl
- [ ] T109 [US6] AI-generate helm/todo-app/templates/NOTES.txt
- [ ] T110 [US6] Record PHRs for helpers/NOTES generation in history/prompts/004-phase4-kubernetes/

### Validation

- [ ] T111 [US6] Lint Helm chart (helm lint ./helm/todo-app) and record output in specs/004-phase4-kubernetes/helm-report.md
- [ ] T112 [US6] Render Helm templates (helm template todo-app ./helm/todo-app --debug) and record output location/notes in specs/004-phase4-kubernetes/helm-report.md
- [ ] T113 [US6] Validate rendered YAML (helm template todo-app ./helm/todo-app | kubectl apply --dry-run=client -f -) and record output in specs/004-phase4-kubernetes/helm-report.md
- [ ] T114 [US6] Fix any Helm template errors (via AI re-prompting) and record changes in specs/004-phase4-kubernetes/helm-report.md
- [ ] T115 [US6] Commit Helm chart under helm/todo-app/ after validation

### Deploy with Helm

- [ ] T116 [US6] Remove raw manifest deployment (kubectl delete -f k8s/ --ignore-not-found=true)
- [ ] T117 [US6] Verify namespace/resources cleaned as expected and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T118 [US6] Install Helm chart with secrets via --set (do not commit secrets) and record the command used (not values) in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T119 [US6] View Helm release (helm list -n todo-app) and record output in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T120 [US6] View post-install notes (helm get notes todo-app -n todo-app) and record output in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T121 [US6] Check Helm release status (helm status todo-app -n todo-app) and record output in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T122 [US6] Verify all resources deployed (kubectl get all -n todo-app) and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T123 [US6] Verify pods running (kubectl get pods -n todo-app) and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T124 [US6] Verify Helm manages all resources (helm get manifest todo-app -n todo-app) and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T125 [US6] Test frontend access (curl http://todo.local) and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T126 [US6] Test backend health (curl http://api.todo.local/health) and record in specs/004-phase4-kubernetes/helm-deploy-report.md
- [ ] T127 [US6] Test complete workflow (register → login → create task → AI chat → logout) and record in specs/004-phase4-kubernetes/helm-deploy-report.md

### Helm operations

- [ ] T128 [US6] Change frontend replica count in helm/todo-app/values.yaml and commit the change
- [ ] T129 [US6] Upgrade Helm release (helm upgrade todo-app ./helm/todo-app -n todo-app) and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T130 [US6] Verify upgrade applied (kubectl get pods -n todo-app) and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T131 [US6] Verify application functional after upgrade and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T132 [US6] Check Helm release history (helm history todo-app -n todo-app) and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T133 [US6] Roll back Helm release (helm rollback todo-app -n todo-app) and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T134 [US6] Verify rollback completed and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T135 [US6] Verify application functional after rollback and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T136 [US6] Update LOG_LEVEL via helm/todo-app/values.yaml and commit the change
- [ ] T137 [US6] Apply config change via helm upgrade and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T138 [US6] Restart deployments to pick up config (kubectl rollout restart ...) and record in specs/004-phase4-kubernetes/helm-ops-report.md
- [ ] T139 [US6] Verify new configuration applied (logs/describe) and record in specs/004-phase4-kubernetes/helm-ops-report.md

**Checkpoint**: Helm-based deployment works and supports upgrade/rollback.

---

## Phase 7: User Story 7 - Documentation, ADRs, and agent/skill docs (Priority: P3)

**Goal**: Phase IV knowledge capture is complete (ADRs, runbooks, prompts catalog, agent/skill docs).

**Independent Test**:
- Required markdown files exist under specs/004-phase4-kubernetes/ and .claude/

### ADRs

- [ ] T140 [P] [US7] Create specs/004-phase4-kubernetes/adr/001-minikube-platform.md
- [ ] T141 [P] [US7] Create specs/004-phase4-kubernetes/adr/002-helm-packaging.md
- [ ] T142 [P] [US7] Create specs/004-phase4-kubernetes/adr/003-ai-generated-infra.md
- [ ] T143 [P] [US7] Create specs/004-phase4-kubernetes/adr/004-stateless-design.md
- [ ] T144 [P] [US7] Create specs/004-phase4-kubernetes/adr/005-nginx-ingress.md

### Documentation

- [ ] T145 [US7] Create specs/004-phase4-kubernetes/architecture.md
- [ ] T146 [US7] Create specs/004-phase4-kubernetes/runbook.md
- [ ] T147 [US7] Create specs/004-phase4-kubernetes/troubleshooting.md
- [ ] T148 [US7] Update README.md with Phase IV deployment instructions
- [ ] T149 [US7] Create specs/004-phase4-kubernetes/ai-prompts-catalog.md
- [ ] T150 [US7] Create final PHR documenting Phase IV completion in history/prompts/004-phase4-kubernetes/

### Agent and skill documentation

- [ ] T151 [P] [US7] Create .claude/agents/infrastructure-generator-agent.md
- [ ] T152 [P] [US7] Create .claude/skills/dockerfile-generation.md
- [ ] T153 [P] [US7] Create .claude/skills/kubernetes-manifest-generation.md
- [ ] T154 [P] [US7] Create .claude/skills/helm-chart-generation.md
- [ ] T155 [US7] Update .claude/README.md to reference Phase IV agents and skills

**Checkpoint**: Docs and ADRs complete.

---

## Phase 8: User Story 8 - Final validation and cleanup (Priority: P3)

**Goal**: Verify AI-generated constraint and operational readiness; produce a final validation report.

**Independent Test**:
- All required reports exist under `specs/004-phase4-kubernetes/`

- [ ] T156 [US8] Verify 100% infrastructure code is AI-generated (git history review) and record evidence in specs/004-phase4-kubernetes/validation-report.md
- [ ] T157 [US8] Verify all AI prompts documented in PHRs and record count in specs/004-phase4-kubernetes/validation-report.md
- [ ] T158 [US8] Verify pods running and healthy (kubectl get pods -n todo-app) and record output in specs/004-phase4-kubernetes/validation-report.md
- [ ] T159 [US8] Verify Phase III features functional (auth, tasks, AI chat) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T160 [US8] Verify performance targets (curl timing checks) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T161 [US8] Verify Helm chart best practices (helm lint) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T162 [US8] Verify scaling works (scale up/down) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T163 [US8] Verify ADRs exist (5 files) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T164 [US8] Verify documentation complete (runbook, troubleshooting, architecture diagram) and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T165 [US8] Remove any test resources from cluster (if present) and record cleanup actions in specs/004-phase4-kubernetes/validation-report.md
- [ ] T166 [US8] Clean up local Docker caches (docker system prune -f) and record action in specs/004-phase4-kubernetes/validation-report.md
- [ ] T167 [US8] Verify /etc/hosts entries for todo.local and api.todo.local and record in specs/004-phase4-kubernetes/validation-report.md
- [ ] T168 [US8] Finalize specs/004-phase4-kubernetes/validation-report.md with a checklist of acceptance criteria and PASS/FAIL for each

**Checkpoint**: Phase IV complete.

---

## Dependencies & Execution Order

### Derived Story Dependencies

- **US1** blocks all other work (cluster and directories must exist)
- **US2** blocks US3/US4 (Dockerfiles must exist to build images; Docker ignore files needed)
- **US3** and **US4** can run in parallel (images vs manifests)
- **US5** depends on US3 + US4 (needs images + manifests)
- **US6** (Helm) can start after US4, but Helm deploy depends on a working cluster and chart validation
- **US7** and **US8** depend on a successful deployment (US5 and/or US6)

### Parallel Opportunities

- US2 frontend and backend Dockerfile work can run in parallel (different directories)
- US4 manifest generation tasks can largely run in parallel (different files)
- US6 Helm templates can run in parallel (different files)
- US7 ADR/doc tasks can run in parallel (different files)

---

## Implementation Strategy

### MVP First (Stop after US5)

1. Complete US1 → US2 → (US3 + US4 in parallel) → US5
2. Validate end-to-end raw-manifest deployment and Phase III functionality in Kubernetes
3. **Stop and demo**

### Incremental Delivery

- Add US6 (Helm chart) after US5 is stable
- Add US7 (docs/ADRs/agent docs)
- Close with US8 final validation and cleanup
