# 📖 Git, GitHub & DevOps - Quick Cheat Sheet
**Bookmark This Page - Read in 15 Minutes**

---

## 🔧 GIT ESSENTIAL COMMANDS

### Basic Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git init                    # Initialize new repo
git clone <url>            # Clone existing repo
```

### Daily Commands
```bash
git status                 # See what changed
git add <file>            # Stage specific file
git add .                 # Stage all changes
git commit -m "message"   # Commit staged changes
git push                  # Push to remote
git pull                  # Fetch + merge from remote
git fetch                 # Download from remote (no merge)
```

### Branching
```bash
git branch                     # List branches
git branch <name>            # Create new branch
git checkout <name>          # Switch to branch
git checkout -b <name>       # Create AND switch
git branch -d <name>         # Delete branch (local)
git push origin <name>       # Push branch to remote
git push origin --delete <name>  # Delete remote branch
```

### Merging & Rebasing
```bash
git merge <branch>           # Merge branch into current
git rebase <branch>          # Replay commits on new base
git merge --abort            # Cancel merge (if conflicts)
git rebase --abort           # Cancel rebase
```

### Undoing Changes
```bash
git restore <file>           # Discard changes in file
git restore --staged <file>  # Unstage file
git reset <file>             # Unstage specific file
git reset --hard HEAD~1      # Undo last commit (DELETE changes!)
git revert HEAD              # Create new commit that undoes changes
git log                       # View commit history
git log --oneline            # Compact commit history
```

### Merge Conflicts
```bash
# When merging fails:
git status                   # See conflicted files
# Edit files manually (look for <<<<<<, ======, >>>>>>)
git add <resolved-file>     # Mark as resolved
git commit -m "Resolved conflicts"
```

---

## 📊 GIT WORKFLOW COMPARISON

### Centralized (Simple)
```
main branch only
↓
All commits to main
↓
No branches
❌ Not recommended for teams
```

### Feature Branch (Recommended)
```
main (stable, production)
  ↓
feature/login branch
  → Work here
  → Create PR
  → Code review
  → Merge to main
  
feature/payment branch
  → Work here
  → Create PR
  → Code review
  → Merge to main

✅ Best for small teams
```

### Git Flow
```
main (production releases)
develop (staging, next release)
  → feature/* (new features)
  → bugfix/* (bug fixes)
  → release/* (release prep)
  → hotfix/* (urgent production fixes)

✅ Best for large teams, multiple releases
```

### Trunk-Based
```
main (always deployable)
  → Short-lived feature branches (1 day max)
  → Frequent PRs and merges
  → Continuous deployment

✅ Best for CI/CD, rapid deployment
```

---

## 💻 GITHUB WORKFLOW

### Creating a Pull Request (PR)
```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes
git add .
git commit -m "Add new feature"

# 3. Push to GitHub
git push origin feature/new-feature

# 4. Create PR on GitHub UI
# → Compare & pull request
# → Add description
# → Request reviewers

# 5. Code review happens
# 6. Merge PR
# 7. Delete branch
```

### Code Review Checklist
- ✅ Does code follow standards?
- ✅ Are there tests?
- ✅ Any security issues?
- ✅ Performance concerns?
- ✅ Is it documented?
- ✅ Does it work?

### PR Best Practices
- Small PRs (easier to review)
- Clear description
- One feature per PR
- Tests included
- No breaking changes without discussion

---

## 📦 DEVOPS TERMINOLOGY

### Environments
```
Development (Dev):
  → What developers work on
  → Frequent changes
  → Latest features
  → Can be broken

Staging:
  → Mirrors production
  → Test before production
  → Run integration tests
  → Performance testing

Production (Prod):
  → Live for users
  → Stable, tested code
  → Real data
  → Careful changes
```

### CI/CD Pipeline
```
Developer pushes code
  ↓
CI (Continuous Integration):
  → Build code
  → Run tests
  → Code quality checks
  → If fails: Notify developer
  ↓
CD (Continuous Deployment):
  → Deploy to staging
  → Run integration tests
  → If success: Deploy to production
  → If fail: Rollback
  ↓
Users get new features/fixes
```

### Deployment Strategies
```
Blue-Green:
  Blue (current live)
  Green (new version, tested)
  Switch all traffic to Green
  Keep Blue for rollback
  ✅ Zero downtime

Rolling Update:
  Replace servers 1-by-1
  10 servers → Update 2, then 2, then 2...
  ✅ No downtime, gradual

Canary:
  Deploy to 5% users first
  Monitor for errors
  If OK: 25% → 50% → 100%
  ✅ Safe, catch bugs early
```

---

## 🐳 DOCKER BASICS

### What is Docker?
```
Problem: Works on my machine, fails on server
Solution: Package entire environment (OS, dependencies, code)
Result: Runs same everywhere

Docker image = Blueprint (like class)
Docker container = Running instance (like object)
```

### Docker Commands
```bash
docker build -t image-name .          # Build image from Dockerfile
docker run -p 3000:3000 image-name   # Run container
  -p port-mapping
  -e ENV_VAR=value (environment variable)
  -d (run in background)

docker ps                             # List running containers
docker logs container-id              # View container logs
docker stop container-id              # Stop running container
docker rm container-id                # Remove container

docker exec container-id bash         # Open shell in container
```

### Dockerfile Example
```dockerfile
FROM node:18                          # Base image
WORKDIR /app                          # Set working directory
COPY . .                              # Copy files
RUN npm install                       # Install dependencies
EXPOSE 3000                           # Expose port
CMD ["npm", "start"]                  # Start command
```

### Docker vs Virtual Machine
```
Virtual Machine:
  → Full OS for each app (2-4 GB each)
  → Slow startup (1-2 mins)
  → Heavy resource usage
  → Complete isolation
  
Docker Container:
  → Shared OS kernel (100s MB each)
  → Fast startup (<1 second)
  → Light resource usage
  → Process-level isolation
```

---

## 🖥️ TERMINAL ESSENTIAL COMMANDS

### Navigation
```bash
pwd                    # Print working directory
ls                     # List files
ls -la                 # List all files with details
cd directory           # Change directory
cd ..                  # Go to parent directory
cd ~                   # Go to home directory
mkdir dirname          # Create directory
rmdir dirname          # Remove empty directory
```

### File Operations
```bash
cp file newfile        # Copy file
mv oldname newname     # Rename/move file
rm file                # Delete file
rm -r directory        # Delete directory
cat file               # View file contents
head -n 10 file        # View first 10 lines
tail -n 10 file        # View last 10 lines
touch filename         # Create empty file
```

### Text Searching
```bash
grep "text" file       # Search for text in file
grep -r "text" .       # Search recursively
grep -v "text" file    # Invert match (exclude)
find . -name "*.js"    # Find files by name
```

### File Permissions
```bash
chmod 755 file         # Change permissions
  7 = read + write + execute (owner)
  5 = read + execute (group)
  5 = read + execute (others)

chown user:group file  # Change owner
```

### Pipes & Redirects
```bash
cat file1 file2 > output   # Redirect to file
cat file1 | grep "text"    # Pipe output to grep
command1 | command2        # Chain commands
```

### Useful Commands
```bash
man command            # Show command manual
which command          # Find command location
ps aux                 # List running processes
top                    # Monitor system resources
df -h                  # Disk space
du -sh directory       # Directory size
env                    # Environment variables
echo $VARIABLE         # Print variable value
```

---

## 🌐 ENVIRONMENT VARIABLES

### What are they?
```bash
Environment variables = Key-value pairs
Used by: Operating system, applications, scripts

Common examples:
PATH = where OS searches for commands
HOME = user's home directory
USER = current user
NODE_ENV = development/staging/production
DATABASE_URL = connection string
```

### Using Environment Variables
```bash
# Set (temporary)
export DATABASE_URL="postgres://localhost/db"

# Use in code (Node.js)
const dbUrl = process.env.DATABASE_URL;

# Permanent (in .env file)
# .env
DATABASE_URL=postgres://localhost/db
NODE_ENV=production
```

### .env File (Never commit!)
```bash
# .env
DATABASE_URL=postgres://localhost/dev_db
API_KEY=secret_key_123
NODE_ENV=development

# Add to .gitignore
echo ".env" >> .gitignore
```

---

## 📊 DEPLOYMENT FLOW

```
Developer
  ↓ git push
GitHub Repository
  ↓ webhook trigger
CI Server (GitHub Actions, Jenkins)
  ↓
  → Clone repo
  → Build code
  → Run tests
  → Build Docker image
  ↓ if tests pass
Push to Registry (Docker Hub, ECR)
  ↓
Deploy Server (AWS, Heroku, Digital Ocean)
  ↓
  → Pull Docker image
  → Run container
  → Health check
  ↓ if healthy
Production Live! ✅
```

---

## 🚨 MONITORING & LOGGING

### Logs
```bash
# View application logs
docker logs container-id
docker logs -f container-id    # Follow logs (live)

# Log levels
ERROR - Something broke
WARN - Something might be wrong
INFO - General information
DEBUG - Detailed troubleshooting
```

### Monitoring
```
Metrics to watch:
  CPU usage
  Memory usage
  Disk space
  Response time
  Error rate
  Request throughput
  
Tools: Prometheus, Grafana, New Relic
```

### Alerts
```
Set up alerts for:
  CPU > 80%
  Memory > 90%
  Error rate > 5%
  Response time > 1 second
  Service down
```

---

## 🔑 KEY DIFFERENCES TO KNOW

### Git vs GitHub
```
Git = Version control system (software)
GitHub = Hosting platform for Git repos
```

### Merge vs Rebase
```
Merge: Combines two branches, creates merge commit, keeps history
Rebase: Replays commits, linear history, cleaner but rewrites history

Use Merge: On main branch, with team
Use Rebase: On feature branch before merging
```

### Staging vs Production
```
Staging: Exact copy of production, tests run here
Production: Live for users, be extra careful
```

### Container vs VM
```
VM: Full OS, heavy, slow, complete isolation
Container: Shared kernel, light, fast, sufficient isolation
```

---

## 💡 QUICK FACTS

- **Commit early, commit often** - Smaller commits easier to revert
- **PR = Pull Request** - How code gets reviewed before merging
- **Main branch** - Should always be production-ready
- **Revert != Reset** - Revert creates new commit, Reset destroys commits
- **Cherry-pick** - Apply specific commit to current branch
- **Stash** - Temporarily save changes without committing
- **Tag** - Mark specific commits (releases)
- **CI/CD** - Automate testing, building, deploying
- **Containerize** - Package app with dependencies
- **Scale up** - More powerful servers (vertical)
- **Scale out** - More servers (horizontal, better)

---

## 🎯 TOP 15 INTERVIEW QUESTIONS

1. **Explain your Git workflow**
2. **Merge vs Rebase - when to use?**
3. **How do you handle merge conflicts?**
4. **What's a Pull Request? Why important?**
5. **Explain CI/CD pipeline**
6. **What is Docker? Why use containers?**
7. **Docker vs Virtual Machine?**
8. **How do you deploy to production?**
9. **Staging vs Production?**
10. **Blue-green deployment?**
11. **Branching strategies?**
12. **How to rollback a bad deployment?**
13. **What are environment variables?**
14. **How do you monitor production?**
15. **Explain Git reset vs revert**

---

**Print this & keep handy! 📌**
