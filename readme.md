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

    # 1. 确保当月目录结构存在
    if [ ! -d "$TARGET_DIR" ]; then
        mkdir -p "$TARGET_DIR"
    fi

    # --- 逻辑分支 ---
    
    if [ -n "$1" ]; then
        # === 模式 A: 快速记录 (带参数) ===
        (
            # 【修改点】：这里加入了 $TODAY，格式变为 "2025-12-20 14:30"
            echo -e "\n- **$TODAY $TIME** $@" >> "$TARGET_FILE"
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
        cd "$TARGET_DIR" || return

        echo "🔄 Pulling updates..."
        git -C "$BASE_DIR" pull --rebase

        # 用 Vim 打开
        vim + "$MONTH.md"

        # 退出 Vim 后，询问是否提交
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
    fi
}
```
