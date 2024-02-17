这是一个备份 GitHub 仓库的 GitHub Action 动作，默认在 push 时触发，可以把你的 GitHub 仓库备份到其它 git 服务器，如 gitee、gitea、gitlab 等

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
    
    - name: set git config
      run: |
        git config --global user.email "${{ secrets.USER_EMAIL }}"
        git config --global user.name "${{ secrets.USER_NAME }}" //配置 git 用户和邮箱
    
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

`${{ secrets.USER_EMAIL }}` 为用户邮箱

`${{ secrets.USER_NAME }}` 为用户名

`${{ secrets.SSH_PRIVATE_KEY }}` 为 ssh 私钥，需要在本地电脑上用 `ssh-keygen` 命令生成，并把私钥上传到仓库 `Setting - Secrets and variables - Actions - Repository secrets` 下，并命名为 `SSH_PRIVATE_KEY`，然后把公钥添加到你目标 git 服务器里

`${{ secrets.GIT_USERNAME }}` 你的目标 git 服务器的登录用户名（如果你是 gitee 用户，则通常是你的手机号）

`${{ secrets.GIT_PASSWORD }}` 你的目标 git 服务器的登录密码
