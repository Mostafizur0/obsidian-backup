https://github.com/kodekloudhub/100-days-of-mlops-solutions/blob/main/Day004%20-%20Add%20a%20.gitignore%20and%20Untrack%20Committed%20Artifacts/solution.md
**About `.gitignore` and untracking:** `.gitignore` tells Git which paths to leave _untracked_, but it has no effect on files Git already tracks. To stop tracking a file that was committed before it was ignored, remove it from the index with `git rm --cached` (which deletes it from Git but leaves the working copy on disk) and commit that change.
##### 1. Inspect what is currently tracked.
Change into the repository and list the tracked files. The listing includes the artifacts that should not be there.
```bash
cd /root/code/fraud-detection
git ls-files
```
##### 2. Create the `.gitignore`.
and add paths that needs to be ignored
##### 3. Untrack the already-committed artifacts.
==`git rm -r --cached` removes the paths from the index only; the files stay in the working tree.==
```bash
git rm -r --cached src/fraud_detection/__pycache__ models/fraud_model.pkl venv .ipynb_checkpoints .env
```
##### 4. Commit the cleanup.
```bash
git add .gitignore
git commit -m "Add .gitignore and untrack committed artifacts"
```
##### 5. Verify.
`git ls-files` now lists only the sources plus `.gitignore`; the artifacts are gone from Git but still present on disk.
```bash
git ls-files
ls -a && ls -a models
```
