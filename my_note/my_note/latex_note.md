## 1、Latex在VS Code 中的设置
可以通过Ctrl+shift+P来查找设置项
```
{
    "[latex]": {
    "editor . formatOnPaste": false,
    "editor . suggestSelection":" recentlyUsedByPrefix"
    },
    "workbench. colorTheme": "Visual Studio Dark" ,

    "window. zoomLevel": 1,
    "files. autoSave": " afterDelay" ,

 // ======================== LaTeX 设置 BEGIN  ========================
        // bibtex 格式
        "latex-workshop.bibtex-format.tab": "tab",
        // 自动编译，全部关闭，当且仅当你认为有需要的时候才会去做编译
        "latex-workshop.latex.autoBuild.run": "never",
        "latex-workshop.latex.autoBuild.cleanAndRetry.enabled": false,
    
        // 设置 latex-workshop 的 PDF　预览程序，external　指的是外部程序
        "latex-workshop.view.pdf.viewer": "external",
        "latex-workshop.view.pdf.ref.viewer": "external",
        "latex-workshop.view.pdf.external.viewer.command": "C:/Users/lenovo/AppData/Local/SumatraPDF/SumatraPDF.exe",
        "latex-workshop.view.pdf.external.viewer.args": [
            "%PDF%"
        ],
    
        // 配置正向、反向搜索：.tex -> .pdf
        "latex-workshop.view.pdf.external.synctex.command": "C:/Users/lenovo/AppData/Local/SumatraPDF/SumatraPDF.exe",
        "latex-workshop.view.pdf.external.synctex.args": [
            // 正向搜索
            "-forward-search",
            "%TEX%",
            "%LINE%",
            "-reuse-instance",
            // 反向搜索
            "-inverse-search",
            "\"D:/Program Files/Microsoft VS Code/Code.exe\" \"D:/Program Files/Microsoft VS Code/resources/app/out/cli.js\" -gr %f:%l",
            "%PDF%"
        ],
    
     // 这是一些独立的编译选项，可以作为工具被编译方案调用
     "latex-workshop.latex.tools": [
        {
            // Windows 原生安装 TeX Live 2020 的编译选项
            "name": "Windows XeLaTeX",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
               // "-pdf",
                "%DOCFILE%"
            ]
        },
        {
            // Windows Biber 编译
            "name": "Windows Biber",
            "command": "biber",
            "args": [
                "%DOCFILE%"
            ]
        },

          // Windows Biber 编译
         { "name": "bibtex",

            "command": "bibtex",
            
            "args": [
            
            "%DOCFILE%"
                ]
      },

        {
            // WSL XeLaTeX 编译一般的含有中文字符的文档
            "name": "WSL XeLaTeX",
            "command": "wsl",
            "args": [
                "/usr/local/texlive/2020/bin/x86_64-linux/xelatex",
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                //"-output-directory=%OUTDIR%",
                //"-aux-directory=%OUTDIR%",
                "%DOCFILE%"
            ]
        },
        {
            // WSL biber / bibtex 编译带有 citation 标记项目的文档
            "name": "WSL Biber",
            "command": "wsl",
            "args": [
                "/usr/local/texlive/2020/bin/x86_64-linux/biber",
                "%DOCFILE%"
            ]
        },
        {
            // macOS 或者 Linux 的简单编译
            // 两种操作系统的操作方式相同
            "name": "macOS / Linux XeLaTeX",
            "commmand": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "%DOCFILE%"
            ]
        },
        {
            // macOS 或者 Linux 的索引编译
            // 两种操作系统的操作方式相同
            "name": "macOS / Linux Biber",
            "command": "biber",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],

    // 这是一些编译方案，会出现在 GUI 菜单里
    "latex-workshop.latex.recipes": [
        {
            // 1.1 Windows 编译简单的小文档，这个选项不太常用，因为绝大多数文章都需要有参考文献索引
            "name": "Windows XeLaTeX 简单编译",
            "tools": [
                "Windows XeLaTeX"
            ]
        },
        {
            // 1.2 Windows 编译带有索引的论文，需要进行四次编译；-> 符号只是一种标记而已，没有程序上的意义
            "name": "Windows xe->bib->xe->xe 复杂编译",
            "tools": [
                "Windows XeLaTeX",
                "Windows Biber",
                "Windows XeLaTeX",
                "Windows XeLaTeX"
            ]
        },
        {
            // 1.2 Windows 编译带有索引的论文，需要进行四次编译；-> 符号只是一种标记而已，没有程序上的意义
            "name": "Windows xe->bibtex->xe 比较复杂编译",
            "tools": [
                "Windows XeLaTeX",
                 "bibtex",
                "Windows XeLaTeX",
                "Windows XeLaTeX"
            ]
        },
       /*  {
            // 2.1  WSL 编译简单的小文档，这个选项不太常用，因为我绝大多数文章都需要有引用。
            "name": "XeLaTeX 简单编译",
            "tools": [
                "WSL XeLaTeX"
            ]
        },
        {
            // 2.2 带有 citation 索引的文档，需要进行四次编译；-> 符号只是一种标记而已，没有程序上的意义
            "name": "xe->bib->xe->xe 复杂编译",
            "tools": [
                "WSL XeLaTeX",
                "WSL Biber",
                "WSL XeLaTeX",
                "WSL XeLaTeX"
            ]
        },
        {
            // 3.1 macOS 简单 小文档
            "name": "macOS XeLaTeX 简单编译",
            "tools": [
                "macOS XeLaTeX"
            ]
        },
        {
            // 3.2 macOS 四次编译
            "name": "macOS xe->bib->xe->xe 复杂编译",
            "tools": [
                "macOS / Linux XeLaTeX",
                "macOS / Linux Biber",
                "macOS / Linux XeLaTeX",
                "macOS / Linux XeLaTeX"
            ]
        } */
    ],
        
    
        // 清空中间文件
        "latex-workshop.latex.autoClean.run": "onBuilt", //注意结尾是 t 不是 d
        "latex-workshop.latex.clean.fileTypes": [
            "*.aux",
            "*.bbl",
            "*.blg",
            "*.idx",
            "*.ind",
            "*.lof",
            "*.lot",
            "*.out",
            "*.toc",
            "*.acn",
            "*.acr",
            "*.alg",
            "*.glg",
            "*.glo",
            "*.gls",
            "*.ist",
            "*.fls",
            "*.log",
            "*.fdb_latexmk",
            "*.bcf",
            "*.run.xml",
          //  "*.synctex.gz"
        ],
        "security.workspace.trust.untrustedFiles": "open",
    // ======================== LaTeX 设置 END ========================
    "latex-workshop.message.error.show": false,
    "latex-workshop.message.warning.show": false,
    "editor.wordWrap": "on",
    "window.zoomLevel": 2,
    "git.path": "D:\\Program Files\\Git\bin\\git.exe",
    
    "terminal.integrated.profiles.windows": {

        "gitBash": {
        
        "path": "D:\\Program Files\\Git\\bin\\bash.exe",
        
        }
        
        },
        "git.confirmSync": false,
        "git.autofetch": true,
    

   // "latex-workshop.showContextMenu": true, //添加LaTex Workshop右键菜单。
   // "latex-workshop.intellisense.package.enabled": true, //根据加载的包，自动完成命令或包。  
  //  "latex-workshop.latex.autoBuild.run": "onSave", //保存文件时自动build（也就是说，点击保存文件或者按快捷键Ctrl+S的时候，除了会保存Tex文件，还会帮你编译LaTex为Pdf。


}
```
## 2、页眉设置设置
```
%==================================设置页眉======================================================
    %     \renewcommand{\headrulewidth}{1pt}  %页眉线宽，设为0可以去页眉线
    % \setlength{\skip\footins}{0.5cm}    %脚注与正文的距离           
    % \renewcommand{\footnotesize}{}      %设置脚注字体大小           
    % \renewcommand{\footrulewidth}{1pt}  %脚注线的宽度               
    %===============                                                
    %双线页眉的设置                                                 
    \pagenumbering{Roman}
    \makeatletter %双线页眉                                        
    \def\headrule{%<!-- -->
    {\if@fancyplain\let\headrulewidth\plainheadrulewidth\fi%
    \hrule\@height 1.5pt \@width\headwidth\vskip1pt%上面线为1pt粗  
    \hrule\@height 0.5pt\@width\headwidth  %下面0.5pt粗            
    \vskip-2\headrulewidth\vskip-1pt}      %两条线的距离1pt        
    \vspace{6mm}
    }     %双线与下面正文之间的垂直间距              
    \makeatother                                                   
    %===============                                                
    \pagestyle{fancy}                                              
    \fancyhead{} %clear all fields                                 
    \fancyhead[CE,CO]{\heiti{插入页眉}}                                                        
    \fancyhead[RO]{\heiti{第 \thepage 页}} %奇数页眉的右边                       
    \fancyhead[LE]{\heiti{第 \thepage 页}} %偶数页眉的左边
    \fancyfoot[LO,RE,CE,CR]{}                      
    \renewcommand{\footrulewidth}{1pt}
    \setlength{\skip\footins}{0.5cm}%脚注与正文的距离 

    \fancypagestyle{plain}{
        \fancyhf{}   
        \fancyfoot[LO,RE]{}   
        \fancyhead[CE,CO]{\heiti{插入页眉}}
        %\fancyhead[RO,LE]{第 \thepage 页} 
        %\fancyhead[LO,RE]{} 
        \fancyhead[RO]{\heiti{第  \thepage 页}}
        \fancyhead[LE]{\heiti{第  \thepage 页}}
        \renewcommand{\footrulewidth}{1pt}
    }
  %==================================设置页眉======================================================
```
## 3、插入公式
```
\begin{equation}
        \label{eq:E1-2-28}
        N_{e f f}=N_{D}+p_{S C}=N_{D}+\frac{J_{p}}{q \cdot E \cdot \mu_{p}(E)}
    \end{equation}
```
## 4、插入图片
```
\begin{figure}[h!] 
        \centering	%使图片居中显示
        \vspace{0.0cm}   %调整图片与上文的垂直距离  
        \setlength{\abovecaptionskip}{0cm} %调整标题上方的距离   
        \setlength{\abovecaptionskip}{0cm} %调整标题下方的距离 	   
        \setlength{\belowdisplayskip}{3pt} 	
        \includegraphics[width=0.5\linewidth]{插入图片名称}
        \caption{ {\zihao{5} 图的标题}} 
        \label{引用标签}
\end{figure}
```
## 5、居中代码
```
\begin{center}  % 居中
    需要居中的设置
\begin{center}  % 居中
```

## 批量注释
<!-- ctrl + / -->
取消同理
