# my log repository

this is to record my thoughts. you can regard it as dairy.

put this script into your `~/.bashrc` or `~/.zshrc` to use it
```bash
function mylog() {
    # --- 配置区域 ---
    # 你的日志仓库根目录
    local BASE_DIR="$HOME/doc/mylogdir"
    
    # 获取时间变量
    local YEAR=$(date +%Y)
    local MONTH=$(date +%Y-%m)
    local TODAY=$(date +%Y-%m-%d)
    local TIME=$(date +%H:%M)
    
    # 目标目录结构: .../2025/
    local TARGET_DIR="$BASE_DIR/$YEAR"
    # 目标文件名: 2025-12.md
    local TARGET_FILE="$TARGET_DIR/$MONTH.md"

    # 0. 确保仓库根目录存在
    if [ ! -d "$BASE_DIR" ]; then
        echo "❌ Error: 目录 $BASE_DIR 不存在，请先 clone 或 mkdir。"
        return 1
    fi

    # 1. 确保当月目录结构存在 (mkdir -p 会自动创建父目录)
    if [ ! -d "$TARGET_DIR" ]; then
        mkdir -p "$TARGET_DIR"
    fi

    # --- 逻辑分支 ---
    
    if [ -n "$1" ]; then
        # === 模式 A: 快速记录 (带参数) ===
        # 使用小括号开启子 shell，执行完自动“返回”原目录，不影响当前终端
        (
            # 追加内容
            echo -e "\n- **$TIME** $@" >> "$TARGET_FILE"
            echo "📝 Logged to $MONTH.md"

            # 进入仓库目录进行 Git 同步
            cd "$BASE_DIR" || exit
            git add .
            git commit -m "Auto: $TODAY $TIME quick log" > /dev/null
            git pull --rebase > /dev/null
            git push > /dev/null
            echo "✅ Synced to Git!"
        )
    else
        # === 模式 B: 深度编辑 (无参数) ===
        # 1. 先进目录
        cd "$TARGET_DIR" || return

        # 2. 打开前先拉取最新代码，防止冲突
        echo "🔄 Pulling updates..."
        git -C "$BASE_DIR" pull --rebase

        # 3. 用 Vim 打开 (参数 + 表示光标直接到文件末尾)
        # 如果文件不存在，vim 会新建一个 buffer，保存时自动创建文件
        vim + "$MONTH.md"

        # 4. 退出 Vim 后，询问是否提交
        echo -n "Push changes to Git? [y/N]: "
        read -r ans
        if [[ "$ans" =~ ^[Yy]$ ]]; then
            git -C "$BASE_DIR" add .
            git -C "$BASE_DIR" commit -m "Manual: $TODAY $TIME edit"
            git -C "$BASE_DIR" push
            echo "✅ Pushed!"
        else
            echo "👌 Local changes saved, not pushed."
        fi
        # 这里不返回原目录，保留在日志目录，方便你继续查看文件
    fi
}
```
