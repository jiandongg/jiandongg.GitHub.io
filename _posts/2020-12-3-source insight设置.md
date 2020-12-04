## 增加多行注释功能及其快捷键

1. 将一下代码保存为word.em文件

```C
macro MultiLineComment()
{
    hwnd = GetCurrentWnd()
    selection = GetWndSel(hwnd)
    LnFirst =GetWndSelLnFirst(hwnd)      //取首行行号
    LnLast =GetWndSelLnLast(hwnd)      //取末行行号
    hbuf = GetCurrentBuf()
    if(GetBufLine(hbuf, 0) =="//magic-number:tph85666031"){
        stop
    }
    Ln = Lnfirst
    buf = GetBufLine(hbuf, Ln)
    len = strlen(buf)
    while(Ln <= Lnlast) {
        buf = GetBufLine(hbuf, Ln)  //取Ln对应的行
        if(buf ==""){                   //跳过空行
            Ln = Ln + 1
            continue
        }
        if(StrMid(buf, 0, 1) == "/"){       //需要取消注释,防止只有单字符的行
            if(StrMid(buf, 1, 2) == "/"){
                PutBufLine(hbuf, Ln, StrMid(buf, 2, Strlen(buf)))
            }
        }
        if(StrMid(buf,0,1) !="/"){          //需要添加注释
            PutBufLine(hbuf, Ln, Cat("//", buf))
        }
        Ln = Ln + 1
    }
    SetWndSel(hwnd, selection)
}
```

2. 将该文件放到project的目录之下的某个地方

3. 点击菜单栏project-add and remove project file，在file name之列双击选择准备好的word.em文件，点击右侧的add按钮。由此，该文件加入了这个项目之中，后续可以在文件搜索栏中搜索到该word.em文件。

4. 点击菜单栏poptions-key assignments，在command栏目中搜索MultiLineComment，随后选中，设置喜欢的快捷键即可。目前我使用的是option+/键的组合。

## 添加项目时，选择add tree即可

## 最好将程序安装在win虚拟机上，配置文件独立管理，项目文件通过网络共享文件夹连接到源代码库里