---
title: "NSIS安装脚本"
description: 使用nsis制作Java程序安装包的记录，nsis还可以直接把jar包生成exe可执行文件，目前使用launch4j来制作exe
tags: [ "安装包脚本程序", "实用工具" ]
categories:
  - "工作总结"
date: 2022-06-01T03:40:46+08:00
draft: false
---
<!--more-->
### 常用变量
{{< admonition type=info title=" 参数脚本" >}}
用户定义的解压路径：
$INSTDIR

程序文件目录：
$PROGRAMFILES

包含 NSIS 安装目录的一个标记。在编译时会检测到。常用于在你想调用一个在 NSIS 目录下的资源时，例如：图标、界面...：
${NSISDIR}

开始菜单目录：
$STARTMENU

开始菜单程序目录：
$SMPROGRAMS
{{< /admonition >}}

### 安装脚本
```nsis
; 该脚本使用 HM VNISEdit 脚本编辑器向导产生

; 安装程序初始定义常量

!define INSTDIR "E:\nsis\nsis_test"
!define PRODUCT_NAME "EFL_PotatoOSp-8601Series"
!define PRODUCT_VERSION "1.2.7-alpha"
!define PRODUCT_PUBLISHER "Engineering For Life"
!define PRODUCT_WEB_SITE "http://www.efl-tech.com/"
!define PRODUCT_DIR_REGKEY "Software\Microsoft\Windows\CurrentVersion\App Paths\${PRODUCT_NAME}.exe"
!define PRODUCT_UNINST_KEY "Software\Microsoft\Windows\CurrentVersion\Uninstall\${PRODUCT_NAME}"
!define PRODUCT_UNINST_ROOT_KEY "HKLM"

SetCompressor lzma

; ------ MUI 现代界面定义 (1.67 版本以上兼容) ------
!include "MUI.nsh"

; MUI 预定义常量
!define MUI_ABORTWARNING
!define MUI_ICON "E:\test\exe\icons\8601P.ico"
!define MUI_UNICON "${NSISDIR}\Contrib\Graphics\Icons\modern-uninstall.ico"

; 欢迎页面
!insertmacro MUI_PAGE_WELCOME
; 安装目录选择页面
!insertmacro MUI_PAGE_DIRECTORY
; 安装过程页面
!insertmacro MUI_PAGE_INSTFILES
; 安装完成页面
!define MUI_FINISHPAGE_RUN "$INSTDIR\${PRODUCT_NAME}.exe"
!insertmacro MUI_PAGE_FINISH
!define MUI_FINISHPAGE_RUN_PARAMETERS "-Dprism.order=sw"

; 安装卸载过程页面
!insertmacro MUI_UNPAGE_INSTFILES
; 安装界面包含的语言设置
!insertmacro MUI_LANGUAGE "SimpChinese"
; 安装预释放文件
!insertmacro MUI_RESERVEFILE_INSTALLOPTIONS
; ------ MUI 现代界面定义结束 ------


;引入nsh脚本-判断系统
!include "x64.nsh"
Name "${PRODUCT_NAME} ${PRODUCT_VERSION}"
OutFile "${PRODUCT_NAME}_setup.exe"
InstallDirRegKey HKLM "${PRODUCT_UNINST_KEY}" "UninstallString"
ShowInstDetails show
ShowUnInstDetails show
BrandingText "EFL, Inc. "
Section "Test" Test
      SetOutPath "$INSTDIR"
SectionEnd
Function .onInit
      #安装目录设置
      ${If} ${RunningX64}
      StrCpy $INSTDIR "$PROGRAMFILES64\${PRODUCT_NAME}"
      ${else}
      StrCpy $INSTDIR "$PROGRAMFILES\${PRODUCT_NAME}"
      ${EndIf}
FunctionEnd

Section "MainSection" SEC01
  SetOverwrite ifdiff
  RMDir /r "$INSTDIR\预置模型"
  RMDir /r "$INSTDIR\update"
  RMDir /r "$INSTDIR\bin\Slic3r"
  RMDIR /r "$SMPROGRAMS\${PRODUCT_NAME}"
  SetOutPath "$INSTDIR\bin"
  File /r "E:\test\exe\bin\*.*"
  SetOutPath "$INSTDIR\image"
  File /r "E:\test\exe\image\*.*"
  SetOutPath "$INSTDIR\jre"
  File /r "E:\test\exe\jre\*.*"
  SetOutPath "$INSTDIR\libs"
  File /r "E:\test\exe\libs\*.*"
  SetOutPath "$INSTDIR\stl"
  File /r "E:\test\exe\stl\*.*"
  SetOutPath "$INSTDIR\update"
  File /r "E:\test\exe\update\*.*"
  SetOutPath "$INSTDIR\预置模型"
  File /r "E:\test\exe\预置模型\*.*"
  SetOutPath "$INSTDIR"
  File "E:\test\exe\${PRODUCT_NAME}.exe"
  ; 创建开始菜单快捷方式
  CreateDirectory "$SMPROGRAMS\${PRODUCT_NAME}"
  CreateShortCut "$SMPROGRAMS\${PRODUCT_NAME}\${PRODUCT_NAME}.lnk" "$INSTDIR\${PRODUCT_NAME}.exe"
  ; 创建桌面菜单快捷方式
  CreateShortCut "$DESKTOP\${PRODUCT_NAME}.lnk" "$INSTDIR\${PRODUCT_NAME}.exe"
SectionEnd

Section -AdditionalIcons
  WriteIniStr "$INSTDIR\EFL官网.url" "InternetShortcut" "URL" "${PRODUCT_WEB_SITE}"
  WriteIniStr "$SMPROGRAMS\${PRODUCT_NAME}\EFL官网.url" "InternetShortcut" "URL" "${PRODUCT_WEB_SITE}"
SectionEnd

Section -Post
  WriteUninstaller "$INSTDIR\uninst.exe"
  WriteRegStr HKLM "${PRODUCT_DIR_REGKEY}" "" "$INSTDIR\${PRODUCT_NAME}.exe"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "DisplayName" "$(^Name)"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "UninstallString" "$INSTDIR\uninst.exe"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "DisplayIcon" "$INSTDIR${PRODUCT_NAME}.exe"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "DisplayVersion" "${PRODUCT_VERSION}"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "URLInfoAbout" "${PRODUCT_WEB_SITE}"
  WriteRegStr ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "Publisher" "${PRODUCT_PUBLISHER}"
SectionEnd

/******************************
 *  以下是安装程序的卸载部分  *
 ******************************/

Section Uninstall
  Delete "$INSTDIR\EFL官网.url"
  Delete "$INSTDIR\uninst.exe"
  Delete "$INSTDIR\${PRODUCT_NAME}.exe"

  Delete "$SMPROGRAMS\${PRODUCT_NAME}\${PRODUCT_NAME}.lnk"
  Delete "$SMPROGRAMS\${PRODUCT_NAME}\EFL官网.url"
  Delete "$DESKTOP\${PRODUCT_NAME}.lnk"

  RMDir /r "$INSTDIR\预置模型"
  RMDir /r "$INSTDIR\update"
  RMDir /r "$INSTDIR\stl"
  RMDir /r "$INSTDIR\jre"
  RMDir /r "$INSTDIR\libs"
  RMDir /r "$INSTDIR\image"
  RMDir /r "$INSTDIR\bin"
  RMDir /r "$SMPROGRAMS\${PRODUCT_NAME}"

  DeleteRegKey ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}"
  DeleteRegKey HKLM "${PRODUCT_DIR_REGKEY}"
  SetAutoClose true
SectionEnd

#-- 根据 NSIS 脚本编辑规则，所有 Function 区段必须放置在 Section 区段之后编写，以避免安装程序出现未可预知的问题。--#
Function .onInstSuccess
  ExecShell "" "$INSTDIR\readme.txt"
FunctionEnd

Function un.onInit
  MessageBox MB_ICONQUESTION|MB_YESNO|MB_DEFBUTTON2 "您确实要完全移除 $(^Name) ，及其所有的组件？" IDYES +2
  Abort
FunctionEnd

Function un.onUninstSuccess
  HideWindow
  MessageBox MB_ICONINFORMATION|MB_OK "$(^Name) 已成功地从您的计算机移除。"
FunctionEnd
```
### 参考链接：
* [NSIS官网文章参考 ](https://nsis.sourceforge.io/A_slightly_better_Java_Launcher)