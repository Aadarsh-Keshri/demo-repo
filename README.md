## Setup Git:
- `sudo apt update`
- `sudo apt upgrade`
- `sudo apt install git -y`
- `git --version`
- `git config --global user.name "GitHub User Name"`
- `git config --global user.email "user email id"`
- `ssh-keygen -t ed25519 -C "user email id"`
- `eval "$(ssh-agent -s)"`
- `ssh-add ~/.ssh/id_ed25519`
- Copy ssh key from: `cat ~/.ssh/id_ed25519.pub`
- Go to GitHub > Settings > SSH and GPG keys.
- Click New SSH Key, give it a title (e.g., "Ubuntu Desktop"), and paste the key.
- Check ssh is connected to GitHub: `ssh -T git@github.com`
- If successful, you will see: "Hi username! You've successfully authenticated..."

## Optimize Git:
1. Command: `git config --global feature.manyFiles true`
   Details:
   This is a "macro" configuration introduced by Git to optimize performance for repositories with hundreds of thousands of files. When you enable this, it toggles several underlying settings:
   * `index.version = 4`: This enables path prefix compression in the Git index file. It makes the index file smaller on your disk, which means less data for your NVMe to read and write during git status or git add.
   * `core.untrackedCache = true`: Git will start using a cache to keep track of untracked files. Instead of scanning every single directory for new files every time you run a command, it looks at the modification time of the directory.
   Performance Impact: For a project like LLVM, which has a massive file tree, this significantly reduces the "lag" you feel when running git status.
   
2. Command: `git config --global index.threads 0`
   Details:
   By default, Git doesn't always use all your CPU cores for operations involving the index (like calculating checksums or searching for changes).
   * The `0` Value: Setting this to 0 tells Git to auto-detect and use all available logical cores on your processor.
   * Leveraging the MultiThreading: Without this setting, Git might only use a single thread for index-heavy tasks. With it, Git can parallelize the workload across all available CPU threads.
   Performance Impact: You will notice the biggest difference during git add . or git commit on large diffs, as the multithreading speeds up the calculation of object hashes.

3. Command: `git config --global core.fsmonitor true`
   Details:
   Since you'll be dealing with many deep directories, you might also want to enable fsmonitor. This allows Git to talk to the Ubuntu kernel to ask "what files changed?" instead of manually checking every file.
   
## Customize Terminal for Git:
1. Locate the Git Prompt Script: `ls /usr/lib/git-core/git-sh-prompt` .If present, we are ready to proceed.
2. `nano ~/.bashrc`
3. Scroll to the very bottom of the file and paste the following block of code:
```sh
  # Git branch integration
if [ -f /usr/lib/git-core/git-sh-prompt ]; then
  . /usr/lib/git-core/git-sh-prompt

  # Optional settings to show more info (1 = enabled)
  export GIT_PS1_SHOWDIRTYSTATE=1      # Shows '*' for unstaged and '+' for staged changes
  export GIT_PS1_SHOWSTASHSTATE=1     # Shows '$' if something is stashed
  export GIT_PS1_SHOWUNTRACKEDFILES=1 # Shows '%' if there are untracked files
  export GIT_PS1_SHOWUPSTREAM="auto"  # Shows '<', '>', '<>', or '=' relative to upstream

  # Customize the prompt (PS1)
  export PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;33m\]$(__git_ps1 "(%s)")\[\033[00m\]\$ '
fi
```

4. Explanation of the above export PS1 line:
   What this does:
   * \[\033[01;32m\]\u@\h: Standard Bold Green for your username and PC name.
   * \[\033[01;34m\]\w: Standard Bold Blue for your folder path.
   * \[\033[01;33m\]$(__git_ps1 "(%s)"): Injects Bold Yellow specifically for the Git branch.
   * \[\033[00m\]\$: Resets everything back to your Theme Default before the $ sign and the text you type.
   
5. `source ~/.bashrc`

