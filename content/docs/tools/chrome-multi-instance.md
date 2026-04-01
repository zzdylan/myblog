---
title: "谷歌浏览器独立多开"
date: 2023-12-26T06:24:00+08:00
lastmod: 2025-05-21T11:56:26+08:00
draft: false
categories: ["工具"]
tags: ["Chrome", "浏览器", "Mac", "Windows"]
---

# mac bash
创建文件
```bash
mkdir chrome_multi_open.sh
```
编写脚本
```bash
#!/bin/bash

# 生成一个基于当前日期和时间的文件名
filename=$(date "+%Y%m%d%H%M%S")

# 创建一个新的Chrome用户数据目录
user_data_dir="./Chrome_User/$filename"

# 检查目录是否已经存在
if [ ! -d "$user_data_dir" ]; then
    mkdir -p "$user_data_dir"
fi


# 创建用户数据目录
mkdir -p "$user_data_dir"

# 启动Google Chrome的新实例
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --user-data-dir="$user_data_dir" --new-window &
```
赋予执行权限
```bash
chmod +x chrome_multi_open.sh
```
执行
```bash
./chrome_multi_open.sh
```

# windows bat批处理
```bash
@echo off
REM 获取日期和时间，格式为YYYYMMDDHHMMSS
set "filename=%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%"
set "filename=%filename: =0%"

REM 确保 Chrome_User 目录存在
if not exist ".\\Chrome_User\\%filename%" mkdir ".\\Chrome_User\\%filename%"

REM 使用唯一的用户数据目录启动新的 Google Chrome 实例
start "" /min "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe" --user-data-dir=".\\Chrome_User\\%filename%" --start-maximized
```

# windows bat批处理2
```bash
@echo off
REM 使用PowerShell获取标准化的时间戳（兼容所有区域设置和新版Windows）
for /f "delims=" %%a in ('powershell -Command "Get-Date -Format 'yyyyMMddHHmmss'"') do set "datetime=%%a"

REM 获取当前脚本所在目录的绝对路径
set "rootdir=%~dp0"

REM 创建用户数据目录（使用绝对路径）
set "userdir=%rootdir%Chrome_User\\%datetime%"
if not exist "%userdir%" mkdir "%userdir%"

REM 启动Chrome（强制使用引号包裹路径）
start "" /min "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe" ^
    --user-data-dir="%userdir%" ^
    --start-maximized ^
    --no-first-run ^
    --disable-features=GlobalMediaControls

echo 已启动Chrome实例，用户数据目录：%userdir%
```
