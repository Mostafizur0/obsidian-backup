[[Git Basics]]

- `git rm -r --cached config.local` : This command removes `config.local` from the Git index (staging area), effectively telling Git to stop tracking it. However, it leaves the `config.local` file untouched in your working directory. After using `git rm --cached`, it's highly recommended to add the filename (`config.local` in this example) to your `.gitignore` file immediately. This prevents you or collaborators from accidentally adding it back into Git tracking in a future commit.
  https://apxml.com/courses/getting-started-with-git/chapter-3-viewing-history-undoing-changes/git-rm-command
- `git ls-files`: This command shows you every file that Git is **actively watching** and keeping track of in your project.
   https://git-scm.com/docs/git-ls-files