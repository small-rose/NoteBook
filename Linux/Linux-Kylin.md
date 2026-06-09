---
layout: default
title: Linux Kylin
has_children: false
parent: Linux
nav_order: 10
---


## 可视化的Nginx启停脚本

```
#!/bin/bash
# Nginx 管理器 - 受限环境专用版（确认弹窗 + 结果提示 + 主窗口常驻）
NGINX_BIN="/data/soft/nginx/sbin/nginx"
CONF="/data/soft/nginx/nginx.conf"
PID_FILE="/data/soft/nginx/logs/nginx.pid"
ERROR_LOG="/data/soft/nginx/logs/error.log"

# 检查 zenity 是否存在
if ! command -v zenity &> /dev/null; then
    echo "❌ 缺少 zenity，无法启动图形界面。" >&2
    exit 1
fi

# 状态检测
get_status() {
    if [ -f "$PID_FILE" ] && kill -0 "$(cat "$PID_FILE" 2>/dev/null)" 2>/dev/null; then
        echo "🟢 运行中 (PID: $(cat "$PID_FILE"))"
    else
        echo "🔴 已停止"
    fi
}

# 执行命令并返回退出码+输出
run_cmd() {
    local cmd="$1"
    local output
    output=$(eval "$cmd" 2>&1)
    echo "$output"
    return $?
}

# 显示确认对话框
show_confirm() {
    local action="$1"
    local desc="$2"
    zenity --question \
        --title="确认操作" \
        --text="确定要【$action】吗？\n\n当前状态：$(get_status)\n\n说明：$desc" \
        --ok-label="是，执行" \
        --cancel-label="取消" \
        2>/dev/null
}

# 显示结果对话框
show_result() {
    local success="$1"
    local msg="$2"
    if [ "$success" = "yes" ]; then
        zenity --info \
            --title="✅ 操作成功" \
            --text="$msg\n\n当前状态：$(get_status)" \
            2>/dev/null
    else
        zenity --error \
            --title="❌ 操作失败" \
            --text="$msg" \
            2>/dev/null
    fi
}

# 主循环：持续显示菜单，直到用户主动退出
while true; do
    # 获取当前状态用于显示
    STATUS_TEXT="$(get_status)"

    ACTION=$(zenity --list --title="Nginx 服务管理器" \
        --text="当前状态：$STATUS_TEXT" \
        --column="操作" --column="说明" \
        "启动" "启动 Nginx 服务" \
        "停止" "安全停止服务" \
        "重启" "完全重启服务" \
        "重载" "平滑重载配置" \
        "日志" "查看最近50行错误日志" \
        --width=420 --height=280 \
        --ok-label="选择此项" \
        --cancel-label="退出程序" \
        2>/dev/null)

    # 用户点击“退出程序”或关闭窗口
    [ -z "$ACTION" ] && exit 0

    # 根据选择执行对应逻辑
    case "$ACTION" in
        启动)
            show_confirm "启动" "将尝试启动 Nginx 服务。若端口被占用或权限不足会失败。" || continue
            OUT=$(run_cmd "$NGINX_BIN -c $CONF")
            [ $? -eq 0 ] && show_result "yes" "Nginx 已成功启动。" || show_result "no" "启动失败：\n$OUT"
            ;;
        停止)
            show_confirm "停止" "将安全停止 Nginx 服务。正在处理的请求会完成后再退出。" || continue
            OUT=$(run_cmd "$NGINX_BIN -c $CONF -s stop")
            [ $? -eq 0 ] && show_result "yes" "Nginx 已安全停止。" || show_result "no" "停止失败：\n$OUT"
            ;;
        重启)
            show_confirm "重启" "将完全重启 Nginx 服务（先停再启）。期间会有短暂中断。" || continue
            OUT=$(run_cmd "$NGINX_BIN -c $CONF -s restart")
            [ $? -eq 0 ] && show_result "yes" "Nginx 已重启。" || show_result "no" "重启失败：\n$OUT"
            ;;
        重载)
            show_confirm "重载" "将平滑重载配置文件，不影响现有连接。适合修改配置后生效。" || continue
            OUT=$(run_cmd "$NGINX_BIN -c $CONF -s reload")
            [ $? -eq 0 ] && show_result "yes" "配置已重载。" || show_result "no" "重载失败：\n$OUT"
            ;;
        日志)
            if [ -f "$ERROR_LOG" ]; then
                zenity --text-info \
                    --title="Nginx 错误日志（最新50行）" \
                    --width=650 --height=450 \
                    2>/dev/null < <(tail -n 50 "$ERROR_LOG")
            else
                zenity --warning \
                    --title="提示" \
                    --text="未找到错误日志文件：\n$ERROR_LOG" \
                    2>/dev/null
            fi
            # 日志查看后不弹结果，直接回到主菜单
            continue
            ;;
    esac
done
```