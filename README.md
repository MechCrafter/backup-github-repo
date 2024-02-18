这是一个备份 GitHub 仓库的 GitHub Action 动作，默认在 push 时触发，可以把你的 GitHub 仓库备份到其它 git 服务器，如 gitee、gitea、gitlab 等

用了几次发现有问题，如果在 GitHub 仓库用了 `git push -f` 命令的话，那这个 action 动作就会运行失败，如果把最后一行的 `git push origin mian` 改成 `git push -f origin mian` 的话也会运行失败，而且用强制推送太不稳妥了，所以不建议用 `git push -f` 命令推送文件到 GitHub 仓库

还有一个问题是 GitHub 是支持 `git lfs` 的，所以你的目标 Git 服务器也要开启 `git lfs`，gitee 没有 `git lfs` 服务，如果你想备份到 gitee 的话就不能开 `git lfs`

由于要把你其它 Git 服务器登录账号密码、ssh 私钥放到 GitHub 服务器里，so 祈祷微软大法好吧，保佑不会有黑客攻击微软服务器并且攻击成功（/doge）

**配置说明**

```yml
name: Backup-github-repo

on: push //默认在 push 时触发

jobs: 
  backup-github-repo:
    runs-on: ubuntu-latest
    steps:
    
    - name: Checkout code
      uses: actions/checkout@v2
      with: 
        fetch-depth: 0
    
    - name: Setup SSH
      run: |
        mkdir -p ~/.ssh/
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan Target-git-server.com >> ~/.ssh/known_hosts //配置 ssh 密钥，此处的 Target-git-server.com 要修改为你目标 git 服务器的主机名
    
    - name: push to Target-git-server.com
      run: |
        echo "Pushing to Target-git-server"
        git remote add origin https://${{ secrets.GIT_USERNAME }}:${{ secrets.GIT_PASSWORD }}@target-git-server.com/${{ secrets.USER_NAME }}/repo.git
        git push origin main //默认 push mian分支
```

`${{ secrets.SSH_PRIVATE_KEY }}` 为 ssh 私钥，需要在本地电脑上用 `ssh-keygen` 命令生成，并把私钥上传到仓库 `Setting - Secrets and variables - Actions - Repository secrets` 下，并命名为 `SSH_PRIVATE_KEY`，然后把公钥添加到你目标 git 服务器里

`${{ secrets.GIT_USERNAME }}` 你的目标 git 服务器的登录用户名（如果你是 gitee 用户，则通常是你的手机号）

`${{ secrets.GIT_PASSWORD }}` 你的目标 git 服务器的登录密码
