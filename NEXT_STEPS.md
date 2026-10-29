# NEXT_STEPS (macOS, beginner friendly)

Everything below marked **YOU RUN** is something you must do yourself.
Nothing here has been run on Jenkins for you: no builds, build numbers, commit IDs or evidence CSVs exist yet.

## What was already prepared in this ZIP (done for you)

| Item | Status |
|---|---|
| `submission.properties` | `class.code=001`, `challenge.code=00488` (both pass the validator in `ci/BuildEvidence.java`) |
| `Jenkinsfile` | Added only the required block `triggers { pollSCM('* * * * *') }`. Nothing else changed. |
| Tests, `ci/*.java`, `src/` | **Untouched.** The starter bug is still present on purpose. |
| `.gitignore` | Original supplied one (ignores `build/`, `data/`, IDE files). |
| Git history / secrets / Jenkins home | None included. |

Verified in my Linux sandbox with JDK 21: the starter compiles and gives **28 tests, 3 failures** (expected red; JAR creation blocked).
Not verified: Jenkins itself, the Git daemon, the agent, macOS behaviour, the green build.

## Which GitHub arrangement?

The assignment's Jenkins job reads `git://127.0.0.1:9418/festpass-java`, served from a **local** repo whose
root contains `Jenkinsfile`. So the graded work happens in a local repo folder named `festpass-java`.
GitHub is just a copy for submission. Recommended: push that local repo to a **separate** GitHub repo
(e.g. `festpass-java`) so history (red -> red -> green commits) is preserved. Adding it as a subfolder of
`cloud-jenkins-lab` also works, but copy files without the `.git` folder and the commit history is lost.
Confirm with your instructor which one they want.

## Steps

### 1. Extract (YOU RUN)
```bash
mkdir -p ~/CloudJenkinsLab && cd ~/CloudJenkinsLab
unzip ~/Downloads/FestPass_Complete_Assignment_001_00488.zip     # creates festpass-java/
unzip ~/Downloads/Jenkins_Laptop_Setup.zip                       # creates jenkins-lab/
ls    # expect: festpass-java  jenkins-lab
```

### 2. Check tools
```bash
java -version; javac -version; git --version     # java and javac must both say 21.x
/usr/libexec/java_home -v 21                     # if not 21, set JAVA_HOME to this path
git daemon --help >/dev/null 2>&1 && echo "git daemon OK"
```
If `git daemon` is missing (Apple's Git sometimes lacks it), install Git with Homebrew: `brew install git`.

### 3. Set Git identity and make the first commit (YOU RUN)
```bash
cd ~/CloudJenkinsLab/festpass-java
git init -b main
git config user.name "vansht24"
git config user.email "your-github-noreply-address"   # use a non-private address
git add .
git commit -m "Configure my Jenkins assignment"
git log -1 --format=%an        # must print: vansht24
```
Do **not** run `java ci/Build.java package` to bypass the failures.
Optional local check: `java ci/Build.java verify` -> expect "28 tests, 3 failures".

### 4. Terminal A - Git server (keep open)
```bash
cd ~/CloudJenkinsLab/jenkins-lab && bash start-git-server.sh
git ls-remote git://127.0.0.1:9418/festpass-java      # from another terminal; expect a hash + refs/heads/main
```

### 5. Terminal B - Jenkins (keep open)
```bash
cd ~/CloudJenkinsLab/jenkins-lab
bash download-jenkins.sh && bash start-jenkins.sh
```
Open http://127.0.0.1:8080/, unlock with `jenkins-lab/runtime/jenkins-home/secrets/initialAdminPassword`,
install suggested plugins, create your admin user, confirm plugins Git / Pipeline / JUnit.

### 6. Agent (Terminal C, keep open)
In Jenkins: Built-In Node executors = 0; new Permanent Agent `student-laptop`, 1 executor, label `student-laptop`,
remote root = your real absolute path ending `/jenkins-lab/runtime/agent-work`, usage "Only build jobs with
label expressions matching this node", launch by connecting to controller (WebSocket). Then:
```bash
cd ~/CloudJenkinsLab/jenkins-lab && bash start-agent.sh      # paste the agent secret at the hidden prompt
```
Never commit or screenshot the secret. The agent terminal must have Java 21 and Git on PATH.

### 7. Create the job
New Item -> `festpass-ci` -> Pipeline -> Pipeline script from SCM -> Git ->
URL `git://127.0.0.1:9418/festpass-java`, credentials none, branch `*/main`, script path `Jenkinsfile`. Save.
Do not add "Build periodically" or any UI trigger.

### 8. Bootstrap build (red expected)
Click **Build Now once**. Expect: Checkout, Submission setup succeed; **Test fails**; Build JAR, Smoke test JAR and
Save artifact skipped; `submission-evidence.csv` archived; no JAR. Open Test Result and Console Output.
If "Submission setup" fails, tell me the console text.

### 9. Automatic red build (YOU RUN)
Edit `src/main/java/FestPass.java`, `usage()` method: change the line
`FestPass - campus event registration` to `FestPass - campus event registration (vansht24)`.
Keep the `add-event` line and all command names unchanged.
```bash
git add src/main/java/FestPass.java
git commit -m "Personalize my help introduction"
git log -1 --oneline
```
Do NOT press Build Now. Wait 1-2 minutes. The new build should say "Started by an SCM change" and fail.
Note its build number and commit ID; compare with `git log -1 --oneline`. Wait for it to finish.

### 10. Diagnose and fix (this is your graded work)
Open each failed test in Jenkins and read the assertion message and the stack trace line. Three tests fail.
Find what they have in common, then read the matching code in `src/main/java/BookingService.java` /
`BookingStore.java` / `FestPass.java`. Make the smallest fix, and write in your own words which business rule was broken.
I deliberately did **not** apply the fix: diagnosis and repair is 30 of the 100 points and must be your own work,
and the red build must exist in your history first. If you get stuck after reading the failures, paste them to me and I will guide you.
```bash
java ci/Build.java verify      # expect 28 tests, 0 failures
java ci/SmokeTest.java         # expect 7 passing smoke checks
git add -A src && git commit -m "<meaningful message about the fix>"
```
Wait for the automatic green build (SUCCESS, 28 tests, 0 failures).

### 11. Evidence (collect only what Jenkins really produced)
From the red (automatic) and green builds download `submission-evidence.csv` into separate folders, e.g.
`evidence/build-N/`. From the green build download `festpass.jar`, then `java -jar festpass.jar help` and show your
`vansht24` message. Make a short evidence page: build numbers, commit IDs, causes, one failing assertion,
skipped stages, test count, JAR output, and a few sentences on what you changed and why.
Do not delete `~/.festpass-assessment/001/installation-id.txt` before grading.

### 12. Push to GitHub (YOU RUN, after creating an empty repo yourself)
```bash
git remote add origin https://github.com/vansht24/<repo-name>.git
git push -u origin main
```
Use your own login/token via the normal GitHub prompt; never put it in a file. Check `git status` shows no `build/` or `data/`.

### 13. Submit
Repository link, red + green evidence CSVs, the final JAR, and the evidence page.

## Requirement checklist and notes

- Job `festpass-ci`, SCM URL/branch/script path, label `student-laptop`: done in the guide, you configure in Jenkins.
- Polling trigger committed: done (`pollSCM('* * * * *')`).
- Codes 001 / 00488: done. Alias `vansht24` comes from the Git author name, not from the properties file (assignment says not to add `student.alias`).
- Conflicting documents: `Student_Lab_Guide.pdf` (8 tests, you introduce `<` to `<=`) is the older lab; the graded guide and
  ASSIGNMENT.md (28 tests, seeded bug, no new bug) are newer, so I followed those.
- Your request wanted a pre-applied fix: not applied, for the reasons in step 10.
