# Jenkins CI/CD Pipeline: Automated Static Website Deployment with Docker and GitHub Webhooks

A complete CI/CD setup on AWS that automatically builds and deploys a static website (Barista Cafe, HTML/CSS/JS) from GitHub to two servers, the Jenkins master and a Jenkins node, using Jenkins, Docker and GitHub Webhooks.

## Architecture

```
Developer ──push──▶ GitHub Repository
                         │  (Webhook, POST on push)
                         ▼
                 Jenkins Master (EC2)
                    │            │
          built-in executor    Jenkins Node (EC2)
                    │            │
            Docker build & run   Docker build & run
                    │            │
            Website on master    Website on node
                 (port 5000)       (port 5000)
```

## Infrastructure (AWS EC2, eu-west-3 Paris)

| Instance | Role |
|---|---|
| `jenkins-master` | Runs Jenkins, receives GitHub webhooks, deploys to itself via the built-in node |
| `node1` | Jenkins agent node, deploys the website via a Jenkins job |
| `developer-server` | Used by the developer to push code |

All instances: `t2.micro`.

## Tools Used

Jenkins 2.516.1, Docker, GitHub (Webhooks), AWS EC2, Git

## Jenkins Jobs

### job1: Freestyle project
- Triggered automatically by the GitHub webhook on every push
- Runs on the built-in (master) server and on the node server
- Builds the Docker image from the repo's `Dockerfile` and serves the site on port 5000
- Result: every push updates the live website without manual steps

### job2: Multi-configuration project
- One job that builds on both the **built-in** server and **node1** in a single run
- Result: both servers serve the same updated website after one push

## Results

| Test | Outcome |
|---|---|
| Edited `index.html` and pushed to GitHub | Webhook delivered successfully, Jenkins build triggered |
| job1 on built-in server | Site updated on master |
| job1 on node server | Site updated on node |
| job2 multi-configuration | Both servers updated together |

Screenshots of each step are in [`kartavya_rathore_DevOps_Project1.pdf`](kartavya_rathore_DevOps_Project1.pdf).

## Repository Structure

```
.
├── Dockerfile
├── index.html
├── reservation.html
├── css/  js/  fonts/  images/  videos/
├── kartavya_rathore_DevOps_Project1.pdf   # Step-by-step screenshots
└── README.md
```

## How to Reproduce

1. Launch 3 EC2 instances (`jenkins-master`, `node1`, `developer-server`) and open the required ports in the security groups (SSH 22, Jenkins 8080, website 5000).
2. Install Jenkins on the master. Install Docker on the master and on the node.
3. Add `node1` as a Jenkins agent.
4. In GitHub: **Settings → Webhooks → Add webhook**, payload URL `http://<jenkins-master-ip>:8080/github-webhook/`, event: push.
5. Create `job1` (Freestyle) with the GitHub repo, enable **GitHub hook trigger for GITScm polling**, and add a build step that builds the Docker image and runs the container on port 5000.
6. Create `job2` (Multi-configuration) with a node axis selecting `built-in` and `node1`.
7. Push a change to `index.html` and watch the sites update.

## Key Learnings

- Triggering builds automatically with GitHub webhooks
- Running the same job on multiple Jenkins nodes
- Containerizing a static website with Docker
- Managing Jenkins master/agent architecture on AWS
