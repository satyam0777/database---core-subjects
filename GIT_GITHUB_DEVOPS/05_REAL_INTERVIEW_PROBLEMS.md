# 🎤 Git, GitHub & DevOps - 15 Real Interview Problems
**With Model Answers & How to Explain**

---

## ✅ PROBLEM 1: Explain Your Git Workflow

### ❓ Question
"Walk me through your typical Git workflow. How do you work with branches, commits, and PRs?"

### 💭 Thought Process
- Interviewer wants real, practical understanding
- Should mention feature branches, PRs, code review
- Show you follow best practices
- Follow-up likely: How handle conflicts, PR review process

### 📋 Expected Answer (90 seconds)

**Setup**:
```
I work in a team on a Node.js/React project
Main branch = production (stable, tested)
```

**My Workflow**:

1. **Start Feature**
   ```bash
   git checkout -b feature/new-login-page
   ```
   - Create feature branch from main
   - Branch name describes what I'm doing
   - Keeps feature isolated

2. **Development**
   ```bash
   # Make changes
   git add .
   git commit -m "Add login form component"
   # More changes
   git add .
   git commit -m "Add password validation"
   # Push to GitHub
   git push origin feature/new-login-page
   ```
   - Commit frequently (small, logical chunks)
   - Each commit should be one logical change
   - Good commits = easy to review, easy to revert

3. **Pull Request**
   - Push branch to GitHub
   - Create PR with description
   - Link to related issues/tickets
   - Request code review from 1-2 teammates

4. **Code Review**
   - Teammate reviews my code
   - Checks: logic, tests, performance, style
   - If issues: Update code locally, push again
   - GitHub auto-updates PR

5. **Merge**
   - Once approved: Merge to main
   - Delete feature branch (clean up)
   - Feature now live after CI/CD passes

6. **Continuous**
   - Feature merged → CI/CD pipeline runs
   - Tests run automatically
   - If pass → deploy to staging
   - If pass staging → deploy to production

**Why This Workflow?**
- **Safety**: Code reviewed before merge
- **Traceability**: Git history shows what changed and why
- **Parallelization**: Multiple features in parallel
- **Rollback**: Easy to revert if issue
- **Collaboration**: Clear who did what

### 🎯 Follow-up Questions (Likely)
1. "What if you need to pull latest main into your feature branch?"
2. "How do you handle merge conflicts?"
3. "What if code review feedback requires major changes?"
4. "How often do you commit?"
5. "What makes a good commit message?"

### 💡 Communication Tips
- ✅ Show practical experience
- ✅ Mention code review (shows collaboration)
- ✅ Explain WHY each step
- ✅ Talk about team coordination
- ✅ Mention best practices you follow
- ❌ Don't just memorize steps
- ❌ Don't forget to mention testing

---

## ✅ PROBLEM 2: Merge vs Rebase - When to Use Each?

### ❓ Question
"Explain the difference between Git merge and rebase. When would you use each? What are the pros and cons?"

### 💭 Thought Process
- This is a technical Git question
- Interviewer tests understanding of version control
- Need to explain both mechanics and philosophy
- Follow-up likely: When safe to rebase, rewriting history

### 📋 Expected Answer (85 seconds)

**Merge**:
```
Main:     A --- B --- C
            \       /
Feature:     D --- E (Merge commit)

Result:   A --- B --- C --- F (merge commit)
            \       /
             D --- E
```

- Creates merge commit combining both branches
- Preserves full history
- Every change visible in history
- If merge breaks: Easy to understand what broke

**Rebase**:
```
Main:     A --- B --- C
Feature:  D --- E

After rebase:  A --- B --- C --- D' --- E'
               (D and E replayed on top of C)
```

- Replays feature commits on top of main
- Linear history (no branching visible)
- Cleaner, easier to read history
- If rebase breaks: Harder to debug

**When to Use Merge**:
1. **Merging into main** (production)
   - Keeps clear record of when feature added
   - Merge commit timestamp = feature ready date
   
2. **Public/shared branches**
   - Other developers might have built on this
   - Don't rewrite their history

3. **Long-lived features**
   - Complex merges documented by merge commit

**When to Use Rebase**:
1. **Before merging PR**
   ```bash
   git fetch origin
   git rebase origin/main  # Update feature with latest main
   git push origin feature --force
   ```
   - Get latest changes from main
   - Linear history for PR review

2. **Local cleanup**
   - Before pushing to GitHub
   - Keep history clean

3. **Feature branch on feature branch**
   - No issue with rewriting history

**Pros/Cons**:

| Aspect | Merge | Rebase |
|--------|-------|--------|
| **History** | Full, non-linear | Linear, clean |
| **Debugging** | Easier (merge commit) | Harder (history rewritten) |
| **Safety** | Safer (doesn't rewrite) | Risky (rewrites history) |
| **For main branch** | ✅ Yes | ❌ No |
| **For feature branch** | ✅ Yes | ✅ Yes (better) |

**Golden Rule**: 
- Merge = Public history (main, shared branches)
- Rebase = Private history (your feature branch before merge)

### 🎯 Follow-up Questions (Likely)
1. "What's force push and when is it safe?"
2. "Can you recover from bad rebase?"
3. "What's an interactive rebase?"
4. "Merge commit vs squash merge?"

### 💡 Communication Tips
- ✅ Draw diagrams showing branching
- ✅ Explain history implications
- ✅ Real use case example
- ✅ Emphasize: Don't rebase public history!
- ❌ Don't go too technical
- ❌ Don't confuse the concepts

---

## ✅ PROBLEM 3: Handling Merge Conflicts

### ❓ Question
"You have a merge conflict. Walk me through how you detect and resolve it."

### 💭 Thought Process
- Practical question about Git problem-solving
- Shows you've dealt with real situations
- Need to explain conflict markers, resolution strategies
- Follow-up: How prevent conflicts, conflict resolution tools

### 📋 Expected Answer (80 seconds)

**Scenario**: Two developers modified same file

**Detection**:
```bash
git merge feature/payment
# ERROR: CONFLICT (content merge) in server.js
# Automatic merge failed; fix conflicts and then commit the result.
```

**Conflict Markers**:
```javascript
const PORT = 3000;
<<<<<<< HEAD
const env = "production";  // My code (main branch)
=======
const env = "staging";     // Feature branch code
>>>>>>> feature/payment
```

**Markers Explain**:
```
<<<<<<< HEAD
  = My current branch code (main)
=======
  = Incoming branch code (feature)
>>>>>>> feature/payment
  = End of conflict
```

**Resolution Steps**:

1. **Understand the conflict**
   ```bash
   git status  # See which files have conflicts
   ```

2. **Open conflicted file**
   - Look at all conflict markers
   - Understand what each side did

3. **Decide how to resolve**
   - Option A: Keep HEAD (main) version
   - Option B: Keep incoming (feature) version
   - Option C: Combine both
   - Option D: Write completely new code

4. **Example resolution**
   ```javascript
   // Decision: Need both values
   const PORT = 3000;
   const env = process.env.NODE_ENV || "production";
   const stagingMode = false;
   ```
   - Remove conflict markers
   - Combine logic that makes sense

5. **Mark resolved**
   ```bash
   git add server.js
   ```

6. **Complete merge**
   ```bash
   git commit -m "Resolve merge conflict in server.js"
   ```

**Prevention Strategies**:

1. **Clear code ownership**
   - Different files for different features
   - Minimize overlapping changes

2. **Communicate**
   - Team knows who's working on what
   - Coordinate if both touching same file

3. **Frequent merges**
   - Keep feature branches short-lived
   - Merge to main often → Less conflict

4. **Smaller PRs**
   - Less code = fewer conflicts
   - Review easier too

5. **Use tools**
   - VS Code shows conflicts nicely
   - IDE merge conflict resolvers

**Manual Resolution**:
```bash
# If really stuck:
git merge --abort  # Cancel merge, start over

# Or:
git checkout --ours file    # Keep main version
git checkout --theirs file  # Keep feature version
```

### 🎯 Follow-up Questions (Likely)
1. "How do you prevent conflicts?"
2. "What's a three-way merge?"
3. "How do IDE helpers work?"
4. "Conflict in binary file?"

### 💡 Communication Tips
- ✅ Show conflict markers clearly
- ✅ Explain resolution options
- ✅ Mention communication prevents issues
- ✅ Real example from your project
- ❌ Don't just abort and force push
- ❌ Don't ignore merge conflicts

---

## ✅ PROBLEM 4: What is CI/CD Pipeline?

### ❓ Question
"Explain CI/CD. What is it and why does it matter? Walk me through your project's pipeline."

### 💭 Thought Process
- This is modern development fundamentals
- Interviewer wants to know you understand automation
- Need to explain both CI and CD parts
- Follow-up: Specific tools, failure handling

### 📋 Expected Answer (90 seconds)

**CI = Continuous Integration**
```
Goal: Integrate code frequently

Process:
1. Developer pushes code
2. CI server automatically:
   - Clones repo
   - Installs dependencies
   - Runs tests
   - Runs linters
   - Checks code quality
3. If all pass: Notify success
4. If fail: Notify failure (developer fixes)

Benefit: Catch bugs early, before merge
```

**CD = Continuous Deployment/Delivery**
```
Goal: Automatic deployment to production

Process:
1. After CI passes
2. CD server automatically:
   - Builds Docker image
   - Deploys to staging
   - Runs integration tests
   - If pass: Deploy to production
   - If fail: Rollback
3. Users get new features immediately

Benefit: Reliable, repeatable deployments
```

**Real Example - My Project**:

```
GitHub (push code)
  ↓
GitHub Actions (CI trigger)
  ↓
  → npm install
  → npm test (runs Jest tests)
  → npm run lint (ESLint checking)
  → npm run build (Next.js build)
  ↓ (if all pass)
  → Build Docker image
  → Push to Docker Hub
  ↓
AWS ECS (CD)
  ↓
  → Pull Docker image
  → Deploy to staging environment
  → Run integration tests
  → Health check: Is app responding?
  ↓ (if healthy)
  → Route traffic to new version (blue-green)
  → Monitor for errors
  ↓
Production Live! ✅
```

**Why It Matters**:

1. **Automation**: Remove human error
2. **Speed**: Deploy 10+ times per day
3. **Safety**: Tests run automatically
4. **Rollback**: If issue, automatic or one-click rollback
5. **Feedback**: Developers know quickly if broke something

**Tools Used**:
- **CI**: GitHub Actions, Jenkins, GitLab CI
- **CD**: AWS CodeDeploy, Heroku, Kubernetes

**Monitoring After Deploy**:
- Error rate monitoring
- Performance metrics
- User feedback
- Logs review
- If issue detected → Immediate rollback

### 🎯 Follow-up Questions (Likely)
1. "What happens if tests fail?"
2. "How do you rollback?"
3. "What's a deployment strategy?"
4. "How often does your pipeline run?"
5. "What metrics do you monitor?"

### 💡 Communication Tips
- ✅ Draw the pipeline flow
- ✅ Explain automation benefits
- ✅ Real project example
- ✅ Mention monitoring
- ✅ Talk about failures and recovery
- ❌ Don't assume they know the tools
- ❌ Don't skip the "why"

---

## ✅ PROBLEM 5: Docker - What & Why?

### ❓ Question
"What is Docker? Why would you use it? What problem does it solve?"

### 💭 Thought Process
- Container fundamentals question
- Need to explain problem first
- Then solution (Docker)
- Compare to alternatives
- Follow-up: Dockerfile, multi-stage builds, networking

### 📋 Expected Answer (80 seconds)

**The Problem** (Before Docker):
```
Developer machine:
  Ubuntu 22.04
  Node 18.2
  PostgreSQL 14
  Redis 7
  Code works perfectly

Server (Production):
  CentOS 7
  Node 16.1 (older!)
  PostgreSQL 12 (older!)
  Redis 5 (older!)

Result: "Works on my machine!" but fails on server
```

**Root Cause**:
```
Environment differences:
  OS version
  Runtime version
  Dependencies
  Configuration
  System libraries
```

**Docker Solution**:
```
Package EVERYTHING:
  Application code
  Runtime (Node 18)
  Dependencies (npm modules)
  OS layer (lightweight Linux)
  Configuration

Result: Same everywhere (dev machine, staging, production)
```

**What is Docker?**
```
Docker = Containerization technology
Container = Lightweight virtual environment with:
  - Only what your app needs
  - Isolated from other apps
  - ~100 MB each (vs 2 GB for VM)
  - Starts in <1 second (vs 1-2 mins for VM)
```

**Docker Image vs Container**:
```
Docker Image = Blueprint (like class definition)
  - Recipe for building container
  - Stored as layers
  - Read-only
  
Docker Container = Running instance (like object)
  - Created from image
  - Running process
  - Can read/write
  - Can be stopped/deleted
```

**Real Example**:

**Dockerfile**:
```dockerfile
FROM node:18                    # Base image
WORKDIR /app                    # Set directory
COPY package*.json .            # Copy deps
RUN npm install                 # Install
COPY . .                        # Copy code
EXPOSE 3000                     # Port
CMD ["npm", "start"]            # Start
```

**Building**:
```bash
docker build -t myapp:1.0 .
```

**Running**:
```bash
docker run -p 3000:3000 myapp:1.0
```

**Result**: 
- Dev runs: `docker run myapp:1.0` → works
- Server runs: `docker run myapp:1.0` → works identical
- Staging runs: `docker run myapp:1.0` → works identical

**Why Companies Use Docker**:

1. **Consistency**: Works everywhere
2. **Deployment**: Quick, reliable
3. **Scaling**: Easy to spin up new containers
4. **Development**: Match production locally
5. **Microservices**: Each service in container

### 🎯 Follow-up Questions (Likely)
1. "How do you pass environment variables?"
2. "Docker vs Virtual Machine?"
3. "How do databases work with Docker?"
4. "What's Docker Compose?"

### 💡 Communication Tips
- ✅ Start with the problem
- ✅ Show Docker as solution
- ✅ Image vs Container distinction clear
- ✅ Real Dockerfile example
- ✅ Practical benefits
- ❌ Don't go too deep into internals
- ❌ Don't skip the problem context

---

## ✅ PROBLEM 6: Deployment Strategies

### ❓ Question
"Explain different deployment strategies. What's blue-green? What's rolling deployment? When use each?"

### 💭 Thought Process
- Advanced DevOps question
- Shows system thinking
- Need to explain tradeoffs
- Follow-up: Canary deployments, health checks

### 📋 Expected Answer (85 seconds)

**The Challenge**:
```
Current: 100 users on v1.0
New: v1.1 has critical features
Problem: How to deploy without downtime?
```

**Strategy 1: Simple Downtime (BAD)**
```
Stop v1.0
Deploy v1.1
Start v1.0
Users: 5-10 mins downtime ❌
```

**Strategy 2: Blue-Green (BEST for quick switch)**
```
Blue (current):   v1.0 live, handling all traffic
Green (new):      v1.1 deployed but idle

Test:             Run integration tests on green
Switch:           Flip switch, all traffic → green
Rollback:         If issue, switch back to blue

Downtime: ZERO ✅
Rollback time: < 10 seconds
Cost: Need 2x servers
```

**Setup**:
```
Load Balancer
  ↓
  Blue Server (v1.0) - 100% traffic
  Green Server (v1.1) - 0% traffic

After tests pass:
  Blue Server (v1.0) - 0% traffic
  Green Server (v1.1) - 100% traffic
```

**Strategy 3: Rolling Update (BEST for resource constraints)**
```
10 servers running v1.0

Step 1: Take down 2 servers
        → Update to v1.1
        → Bring up
        → 8 serving v1.0, 2 serving v1.1

Step 2: Take down 2 more → 6 v1.0, 4 v1.1

Step 3: → 4 v1.0, 6 v1.1

Step 4: → 2 v1.0, 8 v1.1

Step 5: → 0 v1.0, 10 v1.1

Users: No downtime ✅
Rollback: Can still revert mid-deployment
Cost: Only need current servers
```

**Strategy 4: Canary (BEST for risky changes)**
```
100 users on v1.0

Step 1: Deploy v1.1 to 5% of users (5 users)
        → Monitor: errors, latency, user complaints

Step 2: If OK → Deploy to 25% of users

Step 3: If OK → Deploy to 50% of users

Step 4: If OK → Deploy to 100% of users

Benefit: Catch bugs on small % before affecting all
Rollback: Only affected 5% initially
```

**Comparison**:

| Strategy | Downtime | Rollback | Cost | Use Case |
|----------|----------|----------|------|----------|
| **Blue-Green** | ✅ Zero | ✅ Instant | ❌ High | Quick, critical changes |
| **Rolling** | ✅ Zero | ✅ Medium | ✅ Low | General updates |
| **Canary** | ✅ Zero | ✅ Quick | ✅ Low | Risky changes |
| **Simple** | ❌ Downtime | ❌ Slow | ✅ Low | Small services only |

### 🎯 Follow-up Questions (Likely)
1. "How do you handle database migrations?"
2. "What if new version has incompatible API?"
3. "How do you monitor deployment?"
4. "What's shadow traffic?"

### 💡 Communication Tips
- ✅ Draw deployment diagrams
- ✅ Explain zero-downtime importance
- ✅ Compare strategies clearly
- ✅ Mention monitoring during deployment
- ❌ Don't skip the problem context
- ❌ Don't make it too theoretical

---

## Summary - 15 Must-Know Topics

1. ✅ **Git Workflow** - Your development process
2. ✅ **Merge vs Rebase** - When and why
3. ✅ **Merge Conflicts** - Detect and resolve
4. ✅ **CI/CD Pipeline** - Automation and deployment
5. ✅ **Docker Basics** - Containers, images
6. ✅ **Deployment Strategies** - Zero-downtime deployment
7. ✅ **PR/Code Review** - Best practices
8. ✅ **Git Reset vs Revert** - Undoing changes
9. ✅ **Branching Strategies** - Git Flow, trunk-based
10. ✅ **Environment Variables** - Configuration management
11. ✅ **Terminal Commands** - Daily tools
12. ✅ **Monitoring & Logging** - Production visibility
13. ✅ **Rollback Strategies** - Fixing bad deployments
14. ✅ **Staging vs Production** - Environment differences
15. ✅ **Automated Testing** - CI integration

---

**You now have model answers to 15 real interview questions! Practice explaining each in 60-90 seconds. 🚀**

*Continue reading remaining 9 problems in next update...*
