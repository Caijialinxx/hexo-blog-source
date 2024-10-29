---
title: 为了高效同步代码仓库，我写了一个自动化脚本
date: 2024-06-09 15:39:28
tags: [Shell脚本, Git, Notes]
---

> 在日常开发工作中，我经常会遇到需要在两个代码仓库之间批量同步代码的需求。如果手动执行这些同步命令，不仅繁琐，还可能导致出错。而且一旦我需要请假，则要交代其他同事帮忙同步代码，尽管已经写好了教程文档，还是会有各种问题来咨询我，这让我意识到，如果有一个脚本可以帮我完成这些重复性的工作，那该多好，并且同事们拿到脚本只管运行、确认推送就可以了。于是，基于这些目的，我编写了一个自动化脚本，来实现批量代码同步和提交。
> 
> 本文将通过展示脚本的初版和优化版，帮助大家更好地理解如何使用 Bash 脚本实现日常任务的自动化。

## 初版脚本的实现

实现的关键步骤如下：

- 检查当前仓库地址，确认是否与指定地址匹配
- 检查或配置目标仓库地址
- 确认需要同步的分支是否存在
- 同步源仓库代码到目标仓库的对应分支
- 在推送前进行用户确认以防误操作

基于以上步骤，我实现了自动化脚本 `fork-v1.sh`，我们接着来看看初版脚本是如何实现的：

```shell
error_echo() {
  echo -e "\033[31mERROR: $1\033[0m"
}

info_echo() {
  echo -e "\033[34m$1\033[0m"
}

success_echo() {
  echo -e "\033[32m$1\033[0m"
}

cmd_echo() {
  echo -e "\033[33m$1\033[0m"
}

confirm_echo() {
  echo -e "\033[47;35m$1\033[0m"
}

# 检查仓库
REPOSITORY="ssh://git@code.xxx.com/com1/repo1.git"
repository_check_result=$(git remote get-url origin 2>&1)
cmd_echo "$ git remote get-url origin"
echo "$repository_check_result"
if [ "$repository_check_result" == $REPOSITORY ]; then
  success_echo "当前仓库检查通过"
else
  error_echo "当前仓库检查不通过，应进入 $REPOSITORY 对应的本地仓库！"
  exit 1
fi

# 检查源
TARGET_ORIGIN="ssh://git@code.xxx.com/com2/repo2.git"
new_origin_check_result=$(git remote get-url newOrigin 2>&1)
cmd_echo "$ git remote get-url newOrigin"
echo "$new_origin_check_result"
if [ "$new_origin_check_result" == $TARGET_ORIGIN ]; then
  success_echo "当前仓库检查通过"
elif [[ "$new_origin_check_result" =~ "No such remote 'newOrigin'"]]; then
  info_echo "未设置newOrigin，即将自动设置"
  cmd_echo "$ git remote add newOrigin dev"
  git remote add newOrigin dev
  cmd_echo "$ git remote set-url newOrigin $TARGET_ORIGIN"
  git remote set-url newOrigin $TARGET_ORIGIN
else
  error_echo "newOrigin源检查不通过，请根据日志排查失败原因！"
  exit 1
fi

# 检查环境参数
if [ $# -ne 1 ]; then
  error_echo '请传入需要处理的环境参数，例如 fork.sh [dev|uat|prod]'
  exit 1
else
  info_echo "============ 即将执行 repo1 到 repo2 【$1】环境的同步 ============"
fi

# 获取分支名称
if [ $1 == prod ]; then
  OLD_BRANCH_NAME="repo1-prod"
  NEW_BRANCH_NAME="ver"
else
  OLD_BRANCH_NAME="repo1-$1"
  NEW_BRANCH_NAME="repo2-$1"
fi
info_echo "============ 即将执行 repo1 到 repo2 【$1】环境的同步 ============"

# 检查分支
git rev-parse --verify "origin/$OLD_BRANCH_NAME" > /dev/null 2>&1
if [ $? -eq 0 ]; then
  # 分支存在
  success_echo "分支检查通过"
  info_echo "---------- 切换到该分支 ----------"
  cmd_echo "$ git checkout $OLD_BRANCH_NAME"
  git checkout $OLD_BRANCH_NAME
  # 检查是否切换成功
  current_branch=$(git symbolic-ref --short HEAD)
  if [ $current_branch == $OLD_BRANCH_NAME ]; then
    success_echo "已切换到 $OLD_BRANCH_NAME 分支"
  else
    error_echo "分支切换失败，当前分支[$current_branch]"
    exit 1
  fi
else
  error_echo "分支检查失败，原因：该分支不存在，请检查传入的环境参数"
  exit 1
fi

# 同步代码
info_echo "---------- 同步 repo1 仓库 $OLD_BRANCH_NAME 分支的远程代码 ----------"
cmd_echo "$ git fetch origin"
git fetch origin
cmd_echo "$ git reset --hard origin/$OLD_BRANCH_NAME"
git reset --hard origin/$OLD_BRANCH_NAME

info_echo "---------- 拉取 repo2 仓库 $NEW_BRANCH_NAME 分支的远程代码 ----------"
cmd_echo "$ git pull newOrigin $NEW_BRANCH_NAME --allow-unrelated-histories"
git pull newOrigin $NEW_BRANCH_NAME --allow-unrelated-histories

info_echo "---------- 本地 git log 前 5 条 commit 日志简览 ----------"
git log -n 5 --oneline

info_echo "---------- 请人工确认是否推送到远程 repo2 仓库的 $NEW_BRANCH_NAME 分支 ----------"
confirm_echo "输入y为确认推送，q为退出："
while true; do
  read user_input
  case $user_input in
    "y")
      info_echo "---------- 正在推送到远程 repo2 仓库的 $NEW_BRANCH_NAME 分支 ----------"
      git push newOrigin $OLD_BRANCH_NAME:$NEW_BRANCH_NAME
      info_echo "============ 脚本执行完成，请检查是否推送成功 ============"
      break
      ;;
    "q")
      info_echo "停止并退出fork程序"
      exit 0
      ;;
    *)
      confirm_echo "未知指令，请输入'y'或'q'（y为确认推送，q为退出）："
      ;;
  esac
done
```

虽然初版脚本基本满足需求，但在代码结构、容错性和兼容性上还有进一步优化的空间。

## 优化后的脚本

经过优化后的 `fork-v2.sh` 代码结构更加清晰，功能也更加健壮。以下为优化版脚本代码：

```shell
#!/bin/bash

# 定义颜色代码常量
readonly RED='\033[31m'
readonly GREEN='\033[32m'
readonly BLUE='\033[34m'
readonly YELLOW='\033[33m'
readonly MAGENTA_BG_WHITE='\033[47;35m'
readonly NC='\033[0m'  # No Color

error_echo() { printf "${RED}ERROR: %s${NC}\n" "$1"; }
info_echo() { printf "${BLUE}%s${NC}\n" "$1"; }
success_echo() { printf "${GREEN}%s${NC}\n" "$1"; }
confirm_echo() { printf "${MAGENTA_BG_WHITE}%s${NC}\n" "$1"; }
cmd_echo() { printf "${YELLOW}%s${NC}\n" "$1"; }

# 常量定义
readonly REPOSITORY="ssh://git@code.xxx.com/com1/repo1.git"
readonly TARGET_ORIGIN="ssh://git@code.xxx.com/com2/repo2.git"
readonly VALID_ENVS=("dev" "uat" "prod")

# 帮助信息
show_usage() {
  cat << EOF
用法: $(basename "$0") <环境>
环境选项:
  dev   开发环境
  uat   测试环境
  prod  生产环境
示例: 
  $(basename "$0") dev
EOF
  exit 1
}

# 检查环境参数
check_env() {
  if [ $# -ne 1 ]; then
    error_echo "请传入需要处理的环境参数"
    show_usage
  fi
  
  # 使用 =~ 和 IFS 来检查数组是否包含元素
  # 通过在数组元素前后添加空格（" ${VALID_ENVS[*]} "），可以确保准确匹配完整的环境名称，避免部分匹配的问题（比如 "de" 匹配到 "dev"）
  [[ " ${VALID_ENVS[*]} " =~ " $1 " ]] || {
    error_echo "无效的环境参数: $1"
    show_usage
  }
}

# 检查仓库配置
check_repository() {
  cmd_echo "$ git remote get-url origin"

  local repo_url
  repo_url=$(git remote get-url origin 2>&1) || {
    error_echo "获取仓库地址失败"
    exit 1
  }
  
  echo "$repo_url"
  
  if [ "$repo_url" != "$REPOSITORY" ]; then
    error_echo "当前仓库检查不通过，应进入 $REPOSITORY 对应的本地仓库！"
    exit 1
  fi
  success_echo "当前仓库检查通过"
}

# 设置目标仓库
setup_target_repository() {
  cmd_echo "$ git remote get-url newOrigin"

  local target_url
  target_url=$(git remote get-url newOrigin 2>&1)
  
  echo "$target_url"
  
  if [ "$target_url" == "$TARGET_ORIGIN" ]; then
    success_echo "目标仓库检查通过"
    return
  fi
  
  if [[ "$target_url" =~ "No such remote 'newOrigin'" ]]; then
    info_echo "未设置newOrigin，即将自动设置"
    git remote add newOrigin dev && \
    git remote set-url newOrigin "$TARGET_ORIGIN" || {
      error_echo "设置目标仓库失败"
      exit 1
    }
    return
  fi
  
  error_echo "newOrigin源检查不通过，请根据日志排查失败原因！"
  exit 1
}

# 获取分支名称
get_branch_names() {
  if [ "$1" == "prod" ]; then
    OLD_BRANCH_NAME="repo1-prod"
    NEW_BRANCH_NAME="ver"
  else
    OLD_BRANCH_NAME="repo1-$1"
    NEW_BRANCH_NAME="repo2-$1"
  fi
}

# 切换并同步分支
switch_and_sync_branch() {
  # 检查分支是否存在
  git rev-parse --verify "origin/$OLD_BRANCH_NAME" > /dev/null 2>&1 || {
    error_echo "分支 $OLD_BRANCH_NAME 不存在，请检查环境参数"
    exit 1
  }
  
  success_echo "分支检查通过"
  info_echo "---------- 切换到该分支 ----------"
  
  # 切换分支
  git checkout "$OLD_BRANCH_NAME" || {
    error_echo "分支切换失败"
    exit 1
  }
  
  # 验证当前分支
  local current_branch
  current_branch=$(git symbolic-ref --short HEAD)
  if [ "$current_branch" != "$OLD_BRANCH_NAME" ]; then
    error_echo "分支切换失败，当前分支[$current_branch]"
    exit 1
  fi
  success_echo "已切换到 $OLD_BRANCH_NAME 分支"
}

# 主函数
main() {
  check_env "$1"
  check_repository
  setup_target_repository
  get_branch_names "$1"
  
  info_echo "============ 即将执行 repo1 到 repo2 【$1】环境的同步 ============"
  switch_and_sync_branch
  
  # 同步代码
  info_echo "---------- 同步 repo1 仓库 $OLD_BRANCH_NAME 分支的远程代码 ----------"
  git fetch origin && \
  git reset --hard "origin/$OLD_BRANCH_NAME" || {
    error_echo "同步远程代码失败"
    exit 1
  }
  
  # 拉取目标仓库代码
  info_echo "---------- 拉取 repo2 仓库 $NEW_BRANCH_NAME 分支的远程代码 ----------"
  git pull newOrigin "$NEW_BRANCH_NAME" --allow-unrelated-histories || {
    error_echo "拉取目标仓库代码失败"
    exit 1
  }
  
  # 显示日志
  info_echo "---------- 本地 git log 前 5 条 commit 日志简览 ----------"
  git log -n 5 --oneline
  
  # 确认推送
  info_echo "---------- 请人工确认是否推送到远程 repo2 仓库的 $NEW_BRANCH_NAME 分支 ----------"
  confirm_echo "输入y为确认推送，q为退出："
  while true; do
    read -r user_input
    case $user_input in
      "y")
        info_echo "---------- 正在推送到远程 repo2 仓库的 $NEW_BRANCH_NAME 分支 ----------"
        if git push newOrigin "$OLD_BRANCH_NAME:$NEW_BRANCH_NAME"; then
          info_echo "============ 推送成功 ============"
        else
          error_echo "推送失败"
          exit 1
        fi
        break
        ;;
      "q")
        info_echo "停止并退出fork程序"
        exit 0
        ;;
      *)
        confirm_echo "未知指令，请输入'y'或'q'（y为确认推送，q为退出）："
        ;;
    esac
  done
}

main "$@"
```

### 优化点说明

- **代码结构优化**：通过模块化结构，将各功能封装成独立的函数，如 `check_env`、`check_repository`、`switch_and_sync_branch` 等。每个函数负责特定任务，使代码更清晰，也更易于维护和扩展。

- **`echo -e` 替换为 `printf`**：`fork-v2.sh` 中用 `printf` 替换了 `echo -e`，提高了跨平台兼容性。`echo` 的行为在不同的 Unix 系统上可能略有不同，特别是 `-e` 选项的支持情况不一致（如某些系统默认不支持 `-e`），导致转义字符解析可能不一致。`printf` 是 POSIX 标准命令，能保证颜色输出在各种系统中的一致性和格式控制。

- **错误检查和处理机制优化**：优化后的脚本对每一步操作都增加了容错处理。每个关键操作（如仓库地址检查、分支切换、代码同步）都使用 `||` 来保证错误提示和安全退出，防止后续代码在失败条件下继续执行。此外，`switch_and_sync_branch` 函数在切换分支失败时立即退出，避免错误传播，增强了脚本的稳定性。

通过本次优化，脚本在执行批量代码同步时不仅更加稳定，而且具备了良好的容错和提示信息，提升了工作效率。希望本文对你在脚本开发中有所帮助！


---

## 附录

### 常用的颜色与文字样式

#### 文字颜色

| 颜色       | 代码         |
|------------|--------------|
| 黑色       | `\033[30m`   |
| 红色       | `\033[31m`   |
| 绿色       | `\033[32m`   |
| 黄色       | `\033[33m`   |
| 蓝色       | `\033[34m`   |
| 紫色       | `\033[35m`   |
| 青色       | `\033[36m`   |
| 白色       | `\033[37m`   |

#### 背景颜色

| 颜色       | 代码         |
|------------|--------------|
| 黑色背景   | `\033[40m`   |
| 红色背景   | `\033[41m`   |
| 绿色背景   | `\033[42m`   |
| 黄色背景   | `\033[43m`   |
| 蓝色背景   | `\033[44m`   |
| 紫色背景   | `\033[45m`   |
| 青色背景   | `\033[46m`   |
| 白色背景   | `\033[47m`   |

#### 其他常用格式

| 样式       | 代码         |
|------------|--------------|
| 重置所有属性 | `\033[0m`    |
| 高亮/加粗   | `\033[1m`    |
| 下划线     | `\033[4m`    |
| 闪烁       | `\033[5m`    |
| 反显       | `\033[7m`    |

### 特殊参数说明
- `$0`：脚本名称或调用脚本的命令。
- `$1, $2, ...`：脚本接受的第一个、第二个等参数。
- `$#`：传递给脚本的参数个数。
- `$@`：所有参数的数组形式。
- `$*`：所有参数的字符串形式。
- `$$`：当前脚本的进程 ID。
- `$?`：前一个命令的退出状态码。

### `EOF` 用法
`EOF`（End of File）是结束标记，常用于多行字符串输出，还可以替换为其他单词，比如 `END` 、`STOP` 等。两个 `EOF` 之间的所有内容都会被当作文本输出，最后的 `EOF` 必须独占一行且顶格写。
```shell
cat << EOF
多行内容
EOF
```

### `basename "$0"` 用法
`basename "$0"` 提取脚本文件名，不带路径。常用于帮助信息。
```shell
# 例如
basename "/Users/caijialinxx/Desktop/fork.sh"
# 输出: fork.sh

basename "/path/to/file.txt"
# 输出: file.txt
```
