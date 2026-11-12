# PROJECT_CONTEXT.md — PipeLens

> **Living document.** Updated whenever we make a significant change, a decision, or finish a task.
> To resume in a new chat: upload this file and say "Resume from PROJECT_CONTEXT.md".
> **Rule:** nothing counts as implemented unless this file marks it ✅ *confirmed* (the learner pasted output or the files show it). Steps that were explained but not yet confirmed are marked 🟡.

_Last updated: 2026-11-12_

---

## 1. Who and why

| | |
|---|---|
| Learner | Saksham — incoming **Software Engineering Intern, Red Hat** |
| Team | **OpenShift Pipelines** (Red Hat's product built on Tekton) |
| Manager | Navya Tatineni |
| Start date | **Monday, January 11, 2027** |
| Manager's recommended prep | Kubernetes and container basics · Go (go.dev) · React and JavaScript for frontend · Tekton (tekton.dev) |
| Goal | Learn these from basics to advanced by building one real project, and arrive with an edge |

### How lessons are taught (learner preferences)
- Step by step: what to type, where (Windows or Linux), and which folder.
- Every topic covers: **the problem → theory/concept → how it works underneath → why we use it → alternatives → what Red Hat uses → hands-on build → checkpoint (tasks + quiz)**.
- Simple language with analogies. The learner writes the code with guidance; logic is explained, not just dumped.
- The learner keeps personal notes per lesson.
- Teach one lesson at a time and wait for the checkpoint before moving on.

---

## 2. Environment

| Item | Value | Status |
|---|---|---|
| Laptop | Windows 11, 16 GB RAM | ✅ confirmed |
| Linux | WSL2 + **Fedora** (`/etc/os-release` Name = Fedora) | ✅ confirmed |
| WSL memory | ~7.6 GB (default). Plan: raise to ~10 GB in Phase 4 via `.wslconfig` | ✅ confirmed (current value) |
| Go | go1.27.3 linux/amd64 | ✅ confirmed |
| Podman | 6.1.3; `quay.io/podman/hello` ran successfully | ✅ confirmed |
| Git + GitHub SSH | Working; initial commit `c6e635a` pushed | ✅ confirmed |
| VS Code | Windows VS Code + WSL extension (+ Go extension inside WSL) | 🟡 assumed from setup steps |
| Project location | Must be `~/projects/pipelens` (Linux filesystem, **not** `/mnt/d`) | 🟡 move instructions given, not confirmed |
| Podman "/ is not a shared mount" fix | `/etc/wsl.conf` → `[boot] command = "mount --make-rshared /"`, then `wsl --shutdown` | 🟡 given, not confirmed |
| Node.js (for React) | Not installed yet — install in Phase 6 | ⬜ pending |
| GitHub username | Unknown — fill in: `________` | ⬜ open |

---

## 3. Constraints

- **Semester exams: Nov 26 – Dec 2, 2026** → "light mode" only (15–20 min theory reading, no building).
- Other coursework and assignments in November → uneven days; some heavy, some light.
- 16 GB RAM: use **kind** for local Kubernetes. OpenShift Local (CRC) is likely too heavy alongside other apps.
- OpenShift **Developer Sandbox** (free) is namespace-scoped: it probably won't allow installing our own CRDs or cluster-wide RBAC. **Verify in Phase 7.** The design must work without a custom CRD (see §5).
- Avoid Docker Desktop; use Podman (Red Hat's tooling).

---

## 4. The project: PipeLens

**One-liner:** PipeLens watches Tekton PipelineRuns, shows their status, timelines and logs in a dashboard, and explains *why* a run failed.

**Why this project:** it's a small version of what the OpenShift Pipelines team works on: Go code against the Kubernetes API, Tekton controllers and CRDs, and a React/PatternFly UI like the OpenShift console's Pipelines pages. PipeLens is also built and shipped by its own Tekton pipeline ("dogfooding").

### Architecture (target)

```
GitHub repo ──(PR / push)──▶ Pipelines-as-Code ──▶ Tekton Pipeline: test → build (Buildah) → push (Quay) → deploy
                                                                │
Kubernetes cluster (kind locally / OpenShift Sandbox)           ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │ Tekton PipelineRuns / TaskRuns (CRDs)                               │
 │        ▲ watch                                                      │
 │ PipeLens controller (Go, controller-runtime)                        │
 │   - on failure: read TaskRun status + step logs                     │
 │   - analyze (rules first; optional LLM later)                       │
 │   - store result: annotation on PipelineRun (v1)                    │
 │                   own CRD `RunAnalysis` (stretch, kind only)        │
 │        ▲                                                            │
 │ PipeLens API (Go, net/http, JSON)                                   │
 │        ▲                                                            │
 │ PipeLens UI (React + TypeScript + PatternFly, built with Vite)      │
 └─────────────────────────────────────────────────────────────────────┘
```

### Planned repo layout (`~/projects/pipelens`)

```
pipelens/
├── PROJECT_CONTEXT.md     # this file
├── README.md
├── .gitignore
├── cmd/pipelens/          # Go entry point(s)
├── internal/              # Go app code (analyzer, api, controller)
├── deploy/                # Kubernetes/OpenShift YAML
├── tekton/                # Tekton Tasks/Pipelines that build PipeLens
├── web/                   # React frontend
├── scripts/               # helper shell scripts
├── testdata/              # sample logs and fixtures
└── docs/                  # design notes, learning notes, diagrams
```

---

## 5. Decisions log

| Date | Decision | Reason |
|---|---|---|
| 2026-11-09 | Develop on **WSL2 + Fedora**, not native Windows | Containers and Kubernetes are Linux technologies; Red Hat engineers use Fedora/RHEL; Fedora uses the same tools as RHEL (`dnf`, systemd) |
| 2026-11-09 | Use **Podman** instead of Docker | Red Hat's daemonless, rootless container engine |
| 2026-11-10 | Keep code in the Linux filesystem (`~/projects`), not `/mnt/*` | Faster, correct file permissions |
| 2026-11-12 | Team confirmed: **OpenShift Pipelines**; the plan was refocused on K8s/containers, Go, React/JS, Tekton | Manager's email |
| 2026-11-12 | Project re-scoped **KubeLens → PipeLens** (Tekton-focused) | Aligns with the team's product |
| 2026-11-12 | Keep PipeLens (don't restart) but upgrade it with a **Go controller** | Controllers and CRDs are how Tekton itself works; keeps earlier work |
| 2026-11-12 | Failure analysis stored as an **annotation** first; own CRD as a stretch goal | The Developer Sandbox likely blocks custom CRDs |
| 2026-11-12 | Frontend: **React + TypeScript + PatternFly** | Matches the OpenShift console and Red Hat's design system |
| 2026-11-12 | Local cluster: **kind** | Lightweight, fits 16 GB RAM, used by Kubernetes and Tekton upstream CI |

---

## 6. Tech stack: what, why, alternatives, what Red Hat uses

| Area | We use | Why | Alternatives | Red Hat / team context |
|---|---|---|---|---|
| OS | Fedora on WSL2 | Real Linux kernel, same tooling as RHEL | Ubuntu, macOS | Fedora, RHEL |
| Version control | Git + GitHub (SSH) | Industry standard; Tekton is developed on GitHub | GitLab | Upstream work on GitHub (`tektoncd/*`, `openshift-pipelines/*`) |
| Language | Go | Kubernetes, Tekton and most cloud-native tools are written in Go | Rust, Java | Tekton, `tkn`, Pipelines-as-Code, operator are all Go |
| Containers | Podman, Buildah | Daemonless and rootless, so more secure | Docker, containerd/nerdctl | Podman/Buildah/Skopeo; Buildah is common inside Tekton build Tasks |
| Base images | UBI (Universal Base Image) | Free, supported, secure Red Hat base images | Alpine, distroless, Debian | UBI |
| Registry | Quay.io | Hosting images, security scanning | Docker Hub, GHCR | Quay |
| Local Kubernetes | kind | Light, fast, scriptable | minikube, k3d, OpenShift Local (CRC) | OpenShift in production |
| K8s Go libraries | client-go, controller-runtime | Standard ways to talk to the API and write controllers | Raw REST | Tekton uses `knative.dev/pkg` for its controllers (same core ideas) |
| CI/CD | Tekton (+ Triggers, Pipelines-as-Code, Chains) | The team's product | Jenkins, GitHub Actions, GitLab CI, Argo Workflows | **OpenShift Pipelines**; Konflux internally |
| Quick CI on GitHub | GitHub Actions (lint/test) | Easy first CI | — | Many upstream repos use both |
| GitOps (stretch) | Argo CD | Declarative deployments | Flux | OpenShift GitOps |
| Frontend | React + TypeScript + PatternFly, Vite | Matches the OpenShift console | Vue, Angular, MUI | OpenShift console: React + TS + PatternFly |
| Supply-chain security (stretch) | Tekton Chains, cosign | Sign images and record provenance | Notary | Red Hat Trusted Artifact Signer |
| AI (optional stretch) | Local LLM via an OpenAI-compatible API | Plain-English failure explanations | Hosted APIs | OpenShift AI, vLLM |

---

## 7. Schedule and phase status

Legend: ⬜ not started · 🟡 in progress or unconfirmed · ✅ confirmed done

| Phase | Dates (approx.) | Topic | Build | Status |
|---|---|---|---|---|
| 0 | Nov 9–10 | Setup | WSL2, Fedora, Git, Go, Podman, repo | ✅ (except 🟡 items in §2) |
| 1 | Nov 12–16 | Linux + Git | Folder structure, `check-setup.sh`, `analyze-log.sh`, first PR | 🟡 |
| 2a | Nov 17–25 (lighter from Nov 22) | Go basics | Log analyzer CLI in Go + tests | ⬜ |
| — | **Nov 26 – Dec 2** | **EXAMS: light mode** | Reading only: Go Tour, Kubernetes concepts | ⬜ |
| 2b | Dec 3–9 | Go advanced: interfaces, concurrency, context, HTTP, JSON | PipeLens API server | ⬜ |
| 3 | Dec 10–12 | Containers: Podman, Containerfile, UBI, Quay | Container image for PipeLens | ⬜ |
| 4 | Dec 13–20 | Kubernetes: objects, YAML, kind, RBAC, CRDs, controllers, client-go | Deploy on kind; read cluster data | ⬜ |
| 5 | Dec 21–28 | Tekton: Tasks, Pipelines, Workspaces, Results, Triggers, PaC, Chains | Tekton pipeline that builds PipeLens; controller that watches PipelineRuns | ⬜ |
| 6 | Dec 29 – Jan 5 | JavaScript, TypeScript, React, PatternFly | PipeLens dashboard | ⬜ |
| 7 | Jan 6–9 | OpenShift + polish + stretch goals | Deploy on Developer Sandbox; README and demo; a Tekton "good first issue" | ⬜ |
| — | Jan 10 | Rest | — | — |

**Priority if time runs short:** Go → Kubernetes → Tekton → React → OpenShift/stretch.

---

## 8. Lesson progress

| Lesson | Topic | Status | Notes |
|---|---|---|---|
| 0 | Setup | ✅ | Confirmed outputs: go1.27.3, podman 6.1.3, commit `c6e635a` |
| 1.1 | Filesystem and essential commands; repo structure + `.gitignore` | 🟡 taught | Learner hasn't confirmed the structure commit |
| 1.2 | Users, permissions, processes, env vars; `scripts/check-setup.sh` | 🟡 taught | Checkpoint answers not yet received |
| — | Rename KubeLens → PipeLens (GitHub rename, `mv`, `git remote set-url`, `sed`, `web/`, `testdata/`) | 🟡 instructions given | Not confirmed |
| 1.3 | Pipes, grep, text processing; `scripts/analyze-log.sh` + `testdata/pipelinerun.log` | 🟡 taught | Learner paused to finalize the plan; not confirmed |
| 1.4 | Git workflow: branches, commits, rebase, PRs | ⬜ | Next after 1.3 |

---

## 9. Next steps

1. Learner places this file at `~/projects/pipelens/PROJECT_CONTEXT.md` and commits it.
2. Learner runs the **state check** below and pastes the output, so we can mark the 🟡 items ✅ or redo them.
3. Finish Lesson 1.3 (checkpoint), then Lesson 1.4.

### State check command

```bash
cd ~/projects/pipelens 2>/dev/null || echo "NOT FOUND: ~/projects/pipelens"
pwd
git remote -v
git log --oneline
find . -path ./.git -prune -o -type f -print | sort
cat /etc/wsl.conf
```

## 10. Open questions
- GitHub username?
- Were the Lesson 1.1, 1.2 and rename steps completed?
