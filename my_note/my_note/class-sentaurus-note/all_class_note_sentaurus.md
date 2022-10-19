# 2.1 MOSFET建模及基本原理

### **定义变量:**

```scheme{.line-numbers}
(define B 1)
(define pi 3.14)
```

### **更新变量:**

```scheme{.line-numbers}
(set！ B 4)
(set！ i (+ i 1)))
```

### **布尔运算**

"ABA"是新的代替旧的
"BAB"是新的不会代替旧的

```scheme{.line-numbers}

实体不允许交叠!
(sdegeo:set- default-boolean <option> )

```

![布尔运算](/my_note/图片/图片-2.1MOSFET建模及sde操作/布尔运算1.png)

![布尔运算2](/my_note/图片/图片-2.1MOSFET建模及sde操作/布尔运算.png)

获取每个结构的ID号,对其进行操作

```scheme{.line-numbers}

(find-body-id (position x0 y0 z0))
(find-face-id (position x0 y0 z0))
(find-edge-id (position x0 y0 z0))
(get-body-list)
(find-vertex-id (position x0 y0 z0))

(find-region-id (position x0 y0 z0))
(find-material-id (position x0 y0 z0))
```

![结构生成](/my_note/图片/图片-2.1MOSFET建模及sde操作/结构生成.png)

### **倒角定义**

```scheme{.line-numbers}

(sdegeo:fillet-2d (find-vertex-id (position x0 y0 z0) 0.05) #0.05 倒角半径)

```

#### **1.均匀掺杂**

```scheme{.line-numbers}

(sdedr:define-refeval-window  “xxxxxx” 掺杂区域名称 "Rectangle" 
(position <x0> <y0> <z0>) (position <x1> <y1> <z1>))

(sdedr:define-constant-profile (string-append "DC." NAME)
<dopant> <conc>)

(sdedr:define-constant-profile-placement (string-append "CPP.”NAME ) 掺杂变量名
(string-append "DC." DNAME) 掺杂参数名称  (string-append "RW. ”NAME) 掺杂区域名称
[<decay-length>] [<replace>])

(sdedr:define-constant-profile-region 对区域掺杂)
(sdedr:define-constant-profile-material 对材料掺杂)

```

#### **2.高斯掺杂**

掺杂窗口: 一条线/一个面

![高斯掺杂](/my_note/图片/图片-2.1MOSFET建模及sde操作/高斯掺杂.png)

```scheme{.line-numbers}

(sdedr:define-refeval-window "BaseLine.Source" 
	"Line"   #定义掺杂为一条线
	(position (/ W -2) 0.0 0.0)  
	(position (- 0 Wspacer (/ Lgate 2)) 0.0 0.0 ) 
    #线段的两个端点
    )
 
 
(sdedr:define-gaussian-profile "DD.GaussSD" #掺杂类型名称
	"PhosphorusActiveConcentration" #掺杂类型：p型或者n型
	"PeakPos" 0.0  #掺杂浓度峰值位置
    "PeakVal" 1e20  #掺杂浓度峰值
	"ValueAtDepth"  1e15 #结深处的掺杂浓度
    "Depth" 0.1 #结深
	"Gauss"  #掺杂类型：高斯或余误差 详情用户手册
    "Factor" 0.8 #横向扩散定义，一般0.6-0.7
    )

(sdedr:define-analytical-profile-placement "PL.Source" 
	"DD.GaussSD" "BaseLine.Source" 
    	"Positive" # Positive/Negative/Both
        "NoReplace" #“Replace"（全部替换）/"LocalReplace"（替换和这次相同的掺杂）/"NoReplace"（不替换）
        "Eval" #"NoEval"，在基线上是否要赋予掺杂浓度)

```

### **网格定义**

```scheme{.line-numbers}

(sdedr:define-refeval-window 
	"RW.Global" "Rectangle" 
	(position -1000 -1000 0)
	(position  1000  1000 0) 
	)
(sdedr:define-refinement-size "RD.Global" 
	0.1  0.1   #网格最大值
	0.003 0.003 #最小值，满足掺杂浓度变化
    )
 
(sdedr:define-refinement-function "RD.Global" 
   "DopingConcentration" "MaxTransDiff" 1.0 #两个网格点的浓度差异（反双曲正弦函数性质），向外扩展宽度，改小会往外扩展，一般用1
   )

   (sdedr:define-refinement-function "RD.Global" 
	"MaxLenInt" "R.EPI" "R.GateOxide" 1e-3 1.5 #最小值和增长率
	 "DoubleSide" #默认是在前面材料增长，加上后是两侧都进行增长
	 "UseRegionNames" #用区域名代替材料
)

```

![MaxTransDiff解释](/my_note/图片/图片-2.1MOSFET建模及sde操作/maxtransdiff.png)

可以通过msh.cmd查错

得到某个部分ID号，以及查询某个部分的材料和

```scheme{.line-numbers}

(define EPI_BODY_ID (sdegeo:create-rectangle 
	(position (/ W -2) 0 0) 
	(position (/ W 2) Tsi 0) 
	"Silicon" "R.EPI") )

	(generic:get EPI_BODY_ID  "material") #得到材料名字 在.log文件中显示
	(generic:get EPI_BODY_ID  "region") #得到区域名称，在.log文件中显示

```

date：20220714

# 2.2-Trench IGBT的建模与SWB的基本原理（二）

### **预处理阶段判断:**

```scheme{.line-numbers}
; tcl command 
#if 0  #进行注释，1执行，0注释
	#if [string equal "@type@" "NMOS"] 
		(define CHANNEL_DOPING "BoronActiveConcentration")
		(define SD_DOPING "PhosphorusActiveConcentration")
	#else
		(define CHANNEL_DOPING "PhosphorusActiveConcentration")
		(define SD_DOPING "BoronActiveConcentration")
	#endif 
#endif
```

### **SDE里面判断:**

```scheme{.line-numbers}

; scheme command 对就执行第一条，不对执行第二条
(if (string=? "@type@" "NMOS")
	(begin
		(define CHANNEL_DOPING "BoronActiveConcentration")
		(define SD_DOPING "PhosphorusActiveConcentration")
	)
	(begin 
		(define CHANNEL_DOPING "PhosphorusActiveConcentration")
		(define SD_DOPING "BoronActiveConcentration")
	)
)

```

### **网格定义:**

```scheme{.line-numbers}
#只在改变参数的结构附近的网格改变，其他位置的网格不变，保持一致性，便于研究某一参数对器件的影响
(sdesnmesh:axisaligned "xCuts" (list (/ Lgate -2) (/ Lgate 2)))
(sdesnmesh:axisaligned "yCuts" (list (/ Lgate -2) (/ Lgate 2)))

```

### **将命令中的参数变成数字，便于查错（可能存在错误）:**

```scheme{.line-numbers}

(set! evaluate-log-file #t)

```

### **三角函数定义:**

```scheme{.line-numbers}
#其中PI就是软件自带的π
(define TAN (tan (* (/ gate_angle 180) PI) ))
(define COS (cos (* (/ gate_angle 180) PI) ))
(define SIN (sin (* (/ gate_angle 180) PI) )) 

```

### **绘制多边形:**

```scheme{.line-numbers}
#需要形成封闭图形 
(sdegeo:create-polygon (list
  (position Wleft 0.0 0)
  (position (+ Wleft Wgate) 0.0 0)
  (position (- (+ Wleft Wgate) (/ Hgate TAN)) Hgate 0)
  (position (+ Wleft (/ Hgate TAN)) Hgate 0)
  (position Wleft 0.0 0)
  )
  "PolySi" "R.PolyGate")

```

### **列表中取值:**

```scheme{.line-numbers}

>
>(define NUMLIST (list 1 2 3 4 5))
NUMLIST
>(list-ref NUMLIST 0)
> 1
>(list-ref NUMLIST 3)
> 4
>(car NUMLIST)#取列表第一个值
> 1
>(cdr NUMLIST)#取列表除了第一个值
(2 3 4 5)
>

```

### **倒角批量处理:**

```scheme{.line-numbers}
#必须要用car，要不然得到的是一个列表，不是一个数，用不了
(sdegeo:fillet-2d (list 
	(car (find-vertex-id (position (- (+ Wleft Wgate) (/ Hgate TAN)) Hgate 0)))
	(car (find-vertex-id (position (+ Wleft (/ Hgate TAN)) Hgate 0)))
	) R) #R:倒角半径

```

### **合并两个形状并且移动顶点，形成一个形状:**

```scheme{.line-numbers}
#获得第一个形状和ID
(define LOCOS1_ID (sdegeo:create-rectangle
		(position 0.0 0.0 0.0 )  (position Wleft Tox_locos 0.0 ) "Oxide"  "R.LOCOS_thin" ) 
)
#获得第二个形状的ID
(define LOCOS2_ID (sdegeo:create-rectangle
	(position 0.0 (- (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 )  (position Wlocos (+ (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 ) "Oxide"  "R.LOCOS_thick" )
)
#合并两个形状
(define LOCOS_ID (sdegeo:bool-unite (list LOCOS1_ID LOCOS2_ID)) )

(sde:add-material  LOCOS_ID (generic:get LOCOS_ID "material" ) "R.LOCOS") ;只改名称，不改材料
;(sde:add-material  LOCOS_ID  "Gold" (generic:get LOCOS_ID "region" )) #只改材料，不改名称

#移动两个点
(sdegeo:move-vertex
		(list 
			(car
				(find-vertex-id (position Wlocos Tox_locos  0.0 ))
			)
			(car
				(find-vertex-id (position Wlocos 0.0        0.0 ))
			)
		)
		(gvector Wneck 0.0 0.0 ) ;direction length  矢量，包含方向和长度
	)

;倒角设置
(sdegeo:fillet-2d (list 
			(car (find-vertex-id (position Wlocos (+ (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 ) ) )
			(car (find-vertex-id (position Wlocos (- (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 ) ) )
	) Rlocos )

```

### **SDE操作界面:**

```scheme{.line-numbers}
;调用SDE界面
-defaultGUI

```

### **快速修改网格尺寸，通过定义变量实现（不会因器件尺寸导致网格畸变）:**

```scheme{.line-numbers}
(define fs 1.0)
(define dx (* fs (/ (+ Wleft Wgate Wright) 10.0)))
(define dy (* fs (/ ymax 10.0)))
(define dTox (* fs (/ Tox_sidewall 3.0) ))

(sdedr:define-refinement-size "RD.Global" dx dy 1.0  0.1 0.1 0.1 )
(sdedr:define-refeval-window "RW.Global" "Rectangle"  (position -1e5 -1e5 0)  (position 1e5 1e5 0.) )
(sdedr:define-refinement-placement "PL.Global" "RD.Global" "RW.Global" )

```

### **掺杂自定义（.plx文件）:**

```scheme{.line-numbers}

;------read 1d plx doping profile 读取一个1d的掺杂分布曲线
; 1) standard Gauss /Error Function 
; 2) external 1D profile 
; 3) user defined analytical function

;----source doping 
(define Wemitter  0.8)
;----define Emiiter
(sdedr:define-refeval-window "RFW.BaseLineEmitter" "Line"  
  (position (+ Wleft Wgate) 0.0 0.0) (position (+ Wleft Wgate Wemitter) 0.0 0.))

#掺杂参数
(sdedr:define-1d-external-profile "DD.1DExternalEmitter" 
  "n@node|EmitterImp@_fps.plx" "Scale" 1.0 ;是否进行放缩
  "Gauss" ;横向沿什么分布
  "Factor" 0.1 ;横向扩散比例
	 )   

(sdedr:define-analytical-profile-placement "PL.1DExternalEmitter" 
  "DD.1DExternalEmitter" "RFW.BaseLineEmitter" "Both" "NoReplace" "Eval")
 

(sdedr:define-refeval-window "RFW.BaseLinePBody" "Line"  
  (position (+ Wleft Wgate Wspacer) 0.0 0.0) (position (+ Wleft Wgate Wright) 0.0 0.))
(sdedr:define-1d-external-profile "DD.1DExternalPBody" 
  "n@node|BaseImp@_fps.plx" "Scale" 1.0   "Gauss"  "Factor" 0.15)
(sdedr:define-analytical-profile-placement "APL.1DExternalPBody" 
  "DD.1DExternalPBody" "RFW.BaseLinePBody" "Both" "NoReplace" "Eval")






(sdedr:define-refeval-window "BaseLine.fieldstop" "Line"
 (position 0.0 70.0 0.0)
 (position 5.0 70.0 0.0) )
(sdedr:define-gaussian-profile "Impl.fieldstopprof"
 "ArsenicActiveConcentration"
 "PeakPos" 0.0  "PeakVal" 1e19
 "ValueAtDepth" 1e15  "Depth" 3.0
 "Erf" ;[lateral function]横向分布：余误差分布
 "Length" 0.1)
(sdedr:define-analytical-profile-placement "Impl.fieldstop"
 "Impl.fieldstopprof" "BaseLine.fieldstop" "Negative" "NoReplace" "Eval")


```

### **n@node|EmitterImp@，获取lable为EmitterImp的SDE的node号**

**修改载流子寿命:**

1. 先定义基线
2. 物理场名称elifetime，init（初始值），equation（公式），default_value（防止出现数据错误，给一个数来代替）
3. 进行定义
4. note : 1) fieldname must exits in datexcodes.txt file
5. note : 2) 注意正负号+ -，寿命永远大于0

```scheme{.line-numbers}

(sdedr:define-refeval-window "RW.BaselineLifetime" "Line"  
  (position  -1.0  0 0.0) (position (+ Wleft Wgate Wright 1.0) 0 0.0 ))
;												 fieldname    init      equation(eval at every point)   default_value
(sdedr:define-analytical-profile "AP.eLifetime" "eLifetime" "a=5.0;b=5e-6" "1e-5-b*exp(-1.0*x/a)"          0.1     )
; note : 1) fieldname must exits in datexcodes.txt file
;        2) +- 
(sdedr:define-analytical-profile-placement "PL.eLifetime"   "AP.eLifetime" "RW.BaselineLifetime" "Both" "NoReplace" "Eval")
;									 fieldname    init      equation(eval at every point)   default_value
(sdedr:define-analytical-profile "AP.hLifetime" "hLifetime" "a=5.0;b=1e-6" "3e-6-b*exp(-1.0*x/a)"          0.1     )
; note : 1) fieldname must exits in datexcodes.txt file
;        2) +- 
(sdedr:define-analytical-profile-placement "PL.hLifetime"   "AP.hLifetime" "RW.BaselineLifetime" "Both" "NoReplace" "Eval")


```

### **read 1d plx doping profile:**

```scheme{.line-numbers}

;------read 1d plx doping profile
1) standard Gauss/Error Function
2) external 1D profile
3) user defined analytical function|

```

### **offset网格:**

![offset网格](/my_note/图片/图片-2.2-Trench%20IGBT的建模与SWB的基本原理（二）/offset网格.png)

```scheme{.line-numbers}
 
(define nlevels 10) ;10层
(define factor 1.5) ;增长速率

(sdedr:offset-block "material" "Silicon"  "maxlevel" nlevels) ;硅里面的最大层数
;(sdedr:offset-block "region" "R.Si"  "maxlevel" nlevels)

(sdedr:offset-interface "region" "R.Si" "R.Gox" "hlocal" 0.0015 "factor" factor)   ;只对第一层起作用"R.Si"
(sdedr:offset-interface "region" "R.Gox" "R.PolyGate" "hlocal" 0.01 "factor" factor)
(sdedr:offset-interface "region" "R.Gox" "R.Si" "hlocal" (/ Tox_sidewall 5) "factor" factor)

(sdedr:offset-interface "region" "R.Si" "R.LOCOS"  "hlocal" 0.005 "factor" factor)

;需要使用"-offset"来激活offset选项
(sde:build-mesh "-offset" "n@node@")

```

data:20220715

# 2-3 TDR文件的后处理与SJ-LDMOS结构建模

### **高斯掺杂补充:**

```scheme{.line-numbers}

"Erf" {"Factor" lateral-factor | "Length" length}
"Gauss" {"Factor" lateral-factor | "Length" length| "StdDev" standard-deviation}
"Eval" eval-init eval-function

```

### **边界延伸IGBT_extend:**

```scheme{.line-numbers}

(sdegeo:set-default-boolean "BAB")

; step 1 reading boundary of the device
(sdeio:read-tdr-bnd "n@node|IGBT@_bnd.tdr")

;------- get the max/min point in the device 
(define BOX (bbox (get-body-list)))
(display BOX) ; 是一个列表((position min_x min_y min_z) (position max_x max_y max_z))
(define Min_Point (car BOX))
(define Max_Point (cdr BOX)) ;(position max_x max_y max_z)
;car取第一个，cdr取除第一个的值
(define max_y (position:y Max_Point))
(define max_x (position:x Max_Point))
;------- add silicon region in the right
(define Wadd 5.0)
(sdegeo:create-rectangle
  (position max_x 0.0 0.0 )  (position (+ max_x Wadd) max_y 0.0 ) "Silicon"  "R.Add" )

(sdegeo:bool-unite (find-material-id "Silicon"))



; step 2 reading data in the device
(sdedr:define-submesh 
"ExternalProfileDefinition_1"  "n@node|IGBT@_msh.tdr")
; step 3 match (boundary + data)
(sdedr:define-refeval-window "RefEvalWin_1" "Rectangle"  
	Min_Point (position (+ max_x Wadd) max_y 0.0 )
	)
(sdedr:define-submesh-placement "ExternalProfilePlacement_1"  "ExternalProfileDefinition_1" "RefEvalWin_1" "NoReplace" 
	"DecayLength" 1.0)
;DecayLength衰减1微米
; step 4 remesh 
;----mesh defination(simplified)
(sdedr:define-refinement-size "RefinementDefinition_1" 
  1 10 1 0.05 0.05 0.05 )
(sdedr:define-refinement-material "RefinementPlacement_1" 
  "RefinementDefinition_1" "Silicon" )
(sdedr:define-refinement-function "RefinementDefinition_1" 
  "DopingConcentration" "MaxTransDiff" 1)
(sde:build-mesh "n@node@")

```

### **阵列复制IGBT_Array:**

```scheme{.line-numbers}

(sdeio:read-tdr-bnd "n@node|IGBT@_bnd.tdr")
;------- get the max/min point in the device 
(define BOX (bbox (get-body-list)))
(display BOX) ; ((position min_x min_y min_z) (position max_x max_y max_z))
(define Min_Point (car BOX))
(define Max_Point (cdr BOX)) ;(position max_x max_y max_z)

(define max_y (position:y Max_Point))
(define max_x (position:x Max_Point))
(define min_x (position:x Min_Point))
(define min_y (position:y Min_Point))
(define Lx (- max_x min_x))
(define trans (transform:translation (gvector Lx 0.0 0.0)))
(sdegeo:translate-selected (get-body-list) trans #t 4)
;  #t：表示平移后原来边界还在  4：平移四次，得到5个器件
; step 1.2 reading 2rd boundary of the device
;(sdeio:read-tdr-bnd "n@node|IGBT@_bnd.tdr")


(sdegeo:bool-unite (find-material-id "Silicon"))
 
; step 2 reading data in the device
(sdedr:define-submesh "ExternalProfileDefinition_1"  "n@node|IGBT@_msh.tdr")
(sdedr:define-submesh "ExternalProfileDefinition_2"  "n@node|IGBT@_msh.tdr")
; step 3 match (boundary + data)
(sdedr:define-refeval-window "RefEvalWin_1" "Rectangle"  
	Min_Point Max_Point
	)
(sdedr:define-refeval-window "RefEvalWin_2" "Rectangle"  
	(position Lx min_y 0.0) (position (+ Lx Lx) max_y 0.0 )
	)
(sdedr:define-submesh-placement "ExternalProfilePlacement_1"  "ExternalProfileDefinition_1" "RefEvalWin_1" "NoReplace" 
	)
(sdedr:define-submesh-placement "ExternalProfilePlacement_2"  "ExternalProfileDefinition_2" "RefEvalWin_2" "Replace" 
	)
;---- transform submesh 
(sdedr:transform-submesh-placement "ExternalProfilePlacement_2" "ShiftVector" (gvector Lx 0.0 0.0))
 
; step 4 remesh 
;----mesh defination(simplified)
(sdedr:define-refinement-size "RefinementDefinition_1" 
  1 10 1 0.05 0.05 0.05 )
(sdedr:define-refinement-material "RefinementPlacement_1" 
  "RefinementDefinition_1" "Silicon" )
(sdedr:define-refinement-function "RefinementDefinition_1" 
  "DopingConcentration" "MaxTransDiff" 1)


(sde:build-mesh "n@node@")

```

### **对称IGBT_Mirror:**

```scheme{.line-numbers}

;cmd命令行
tdx -mtt -x(/-X) -ren "Gate=Gate" n6_msh.tdr n6_mirrored_msh.tdr
;-x(/-X  -y -Y),小写x表示x轴最小值的地方对称，大写X表示x轴最大值的地方对称
;n6_msh.tdr 用来对称的tdr文件
;n6_mirrored_msh.tdr 形成的tdr文件
;-ren "Gate=Gate"，-ren重新命名
;没有remesh，网格严格对称
```

### **Cshell工具:**

```scheme{.line-numbers}
;执行终端中的命令
tdx -mtt -X -ren "Gate=Gate" n@previous@_msh.tdr n@previous@_mirrored_msh.tdr 

```

### **拉伸三维IGBT_sweep:**

```scheme{.line-numbers}

; Using DF-ISE coordinate system for structure generation
(sde:set-process-up-direction "+z")
;----------------------------------------------------------------------
; Structure definition
;----------------------------------------------------------------------;
(sdegeo:set-default-boolean "BAB")
; step 1 reading boundary of the device
(sdeio:read-tdr-bnd "n@node|IGBT@_bnd.tdr")

;------- get the max/min point in the device 
(define BOX (bbox (get-body-list)))
(display BOX) ; ((position min_x min_y min_z) (position max_x max_y max_z))
(define Min_Point (car BOX))
(define Max_Point (cdr BOX)) ;(position max_x max_y max_z)

(define max_y (position:y Max_Point))
(define max_x (position:x Max_Point))

;------- sweep the structure
(sdegeo:sweep (get-body-list) 10.0  (sweep:options "solid" #t  "rigid" #f "miter_type" "default"))
;10 向z轴扩展10微米 

; step 2 reading data in the device
(sdedr:define-submesh "ExternalProfileDefinition_1"  "n@node|IGBT@_msh.tdr")
; step 3 match (boundary + data)
(sdedr:define-refeval-window "RefEvalWin_1" "Rectangle"  
	Min_Point Max_Point
	)
(sdedr:define-submesh-placement "ExternalProfilePlacement_1"  "ExternalProfileDefinition_1" "RefEvalWin_1" "NoReplace" 
	"DecayLength" 1.0)


;-----sweep the Ref/Eval Window 
(sdegeo:sweep (get-drs-list) 10.0  (sweep:options "solid" #t  "rigid" #f "miter_type" "default"))
 

; step 4 remesh 
;----mesh defination(simplified)
(sdedr:define-refinement-size "RefinementDefinition_1" 
  1 10 4 1 1 1 )
(sdedr:define-refinement-material "RefinementPlacement_1" 
  "RefinementDefinition_1" "Silicon" )
(sdedr:define-refinement-function "RefinementDefinition_1" 
  "DopingConcentration" "MaxTransDiff" 1)

(sde:build-mesh "n@node@")

```

### **旋转三维IGBT_sweep_angle:**

```scheme{.line-numbers}

; Using DF-ISE coordinate system for structure generation
(sde:set-process-up-direction "+z")
;----------------------------------------------------------------------
; Structure definition
;----------------------------------------------------------------------;
(sdegeo:set-default-boolean "BAB")
; step 1 reading boundary of the device
(sdeio:read-tdr-bnd "n@node|IGBT@_bnd.tdr")

;------- get the max/min point in the device 
(define BOX (bbox (get-body-list)))
(display BOX) ; ((position min_x min_y min_z) (position max_x max_y max_z))
(define Min_Point (car BOX))
(define Max_Point (cdr BOX)) ;(position max_x max_y max_z)

(define max_y (position:y Max_Point))
(define max_x (position:x Max_Point))
;------- sweep the structure
;(sdegeo:sweep (get-body-list) 10.0  (sweep:options "solid" #t  "rigid" #f "miter_type" "default"))
(sdegeo:sweep (get-body-list) (position 0  0  0) 旋转点(gvector  0 -1 0) 旋转轴
   (sweep:options "solid" 是否立体 #t "sweep_angle" 270 "rigid" #f "miter_type" 
     "default")) 
(map bool:regularise (get-body-list)) 做一个规范化处理，减少网格

; step 2 reading data in the device
(sdedr:define-submesh "ExternalProfileDefinition_1"  "n@node|IGBT@_msh.tdr")
; step 3 match (boundary + data)
(sdedr:define-refeval-window "RefEvalWin_1" "Rectangle"  
	Min_Point Max_Point
	)
(sdedr:define-submesh-placement "ExternalProfilePlacement_1"  "ExternalProfileDefinition_1" "RefEvalWin_1" "NoReplace" 
	"DecayLength" 1.0)


;-----sweep the Ref/Eval Window 
(sdegeo:sweep (get-drs-list) (position 0  0  0) (gvector  0 -1 0) 
   (sweep:options "solid" #t "sweep_angle" 270 "rigid" #f "miter_type" 
     "default")) 
 
; step 4 remesh 
;----mesh defination(simplified)
(sdedr:define-refinement-size "RefinementDefinition_1" 
    1 10 1 0.05 0.05 0.05 )
(sdedr:define-refinement-material "RefinementPlacement_1" 
  "RefinementDefinition_1" "Silicon" )
(sdedr:define-refinement-function "RefinementDefinition_1" 
  "DopingConcentration" "MaxTransDiff" 1)
 
(sde:build-mesh "n@node@")

```

### **创建长方体:**

```scheme{.line-numbers}

(sdegeo:create-cuboid position(x0 y0 z0) position(x1 y1 z1))
创建长方体

```

### **字符串连接:**

```scheme{.line-numbers}

(define NAME "Nn_plus")
(sdedr:define-analytical-profile-placement (string-append "IMP." NAME) 
	(string-append "GAUSS." NAME) (string-append "RW." NAME) 
	"Negative"  "NoReplace" "Eval")
(string-append "GAUSS." NAME)
字符串的连接得到 GAUSS.Nn_plus

```

### **更新变量:**

```scheme{.line-numbers}

(set! NAME "Drain") ;update variables using set!

```

### **保存界面:**

```scheme{.line-numbers}

(sde:build-mesh "-AI"  "n@node@_msh")
;加上 "-AI"

```

date：20220716

# 2-4-FinFET_Nanosheet器件建模与随机涨落方法

### **镜像IGBT作业补充:**

```scheme{.line-numbers}

(define trans (transform:reflection (position max_x 0.0 0.0 ) (gvector 1.0 0.0 0.0) ) )
(define single_device (get-body-list))
(sdegeo:mirror-selected single_device trans #t)
;把所选进行器件镜像，沿什么镜像，对称轴，对称轴在哪个点

(sdegeo:bool-unite (find-material-id "Silicon"))

; step 2 reading data in the device
(sdedr:define-submesh "ExternalProfileDefinition_1" "n@node|IGBT@_msh.tdr")
(sdedr:define-submesh "ExternalProfileDefinition_2"  "n@node|IGBT@_msh.tdr")


;step 3 match (boundary + data)
(sdedr:define-refeval-window "RefEvalWin_1" "Rectangle"
Min_Point Max_Point)

(sdedr:define-refeval-window "RefEvalWin_2" "Rectangle"
(position Lx min_y 0.0) (position (+ Lx Lx) max_y 0.0 ))

(sdedr:define-submesh-placement "ExternalProfilePlacement_1"
"ExternalProfileDefinition_1" "RefEvalWin_1" "NoReplace")

(sdedr:define-submesh-placement "ExternalProfilePlacement_2" "ExternalProfileDefinition_2" "RefEvalWin_2" "Replace")

;transform submesh
(sdedr:transform-submesh-placement "ExternalProfilePlacement_2" "Reflect" "X"
 "ShiftVector" (gvector Lx 0.0 0.0))
;"Reflect" "X"沿x轴原地翻转180度
;"ShiftVector" (gvector Lx 0.0 0.0) 平移
```

### **给边做倒角:**

```scheme{.line-numbers}
;一般选边的终点
(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) ptop_right_y (+ z0 h ))) r_rounding_top)
(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) ptop_left_y  (+ z0 h ))) r_rounding_top)

```

### **定义函数(匿名函数):**

![定义函数](/my_note/图片/2.4/定义函数.png)

```scheme{.line-numbers}

(define create_trapzoid 
	(lambda (x0 y0 z0 w h l theta rmater rname r_rounding_top r_rounding_bot direction) 

		(define sin_theta (sin ( / (* theta PI ) 180) ) )
		(define cos_theta (cos ( / (* theta PI ) 180) ) )
		         
		(define pbot_left_y  (/ w -2) )
		(define pbot_right_y (/ w  2) )

		(define pdy (* h (/ cos_theta sin_theta)  ) )

		(define ptop_left_y  (+ pbot_left_y  pdy ) )
		(define ptop_right_y (- pbot_right_y pdy ) )

		(define polygon_list (list 
								 (position x0 pbot_left_y  z0)
								 (position x0 pbot_right_y z0)
								 (position x0 ptop_right_y (+ z0 h))
								 (position x0 ptop_left_y  (+ z0 h))
							 )
		)

		(define polygon_1 (sdegeo:create-polygon polygon_list rmater rname))

        		##--sweep polygon
		(sdegeo:sweep polygon_1 (gvector (* l direction) 0 0 ) 
		(sweep:options "solid" #t "rigid" #f "miter_type" "default"))

		(if (> r_rounding_top 0)
			(begin
				(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) ptop_right_y (+ z0 h ))) r_rounding_top)
				(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) ptop_left_y  (+ z0 h ))) r_rounding_top)
			)
		); endif

		(if (> r_rounding_bot 0 )
			(begin
				(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) pbot_right_y  z0)) r_rounding_bot)
				(sdegeo:fillet (find-edge-id (position (/ (+ x0 x0 (* direction l)) 2) pbot_left_y   z0)) r_rounding_bot)
			)
		);endif
	);lambda
); function end 

(create_trapzoid  0  0  0   Wfin  Hfin  Lgate  FinAngle "Silicon" "R.channel" Rfin 0 1)


```

### **循环语句:**

```scheme{.line-numbers}

(do
	((i 1 (+ i 1)))
	((> i 10))
		(commands1...)
		(commands2...)
		(commands3...)

```

### **循环例子:**

```scheme{.line-numbers}

(do 
	; init variable
	(
		(i 1 (+ i 1))
	)
	; stop criterial
	(
		(> i Num)
	)
	(sdegeo:create-rectangle (position x0 y0 0.0)
		(position (+ x0 W) (+ y0 L) 0.0) 
		"Silicon" (string-append "R.Silicon" (number->string i)))
	(set! x0 (+ x0 dx W))
); do 

(do 
	; init variable
	(
		(i 1 (+ i 1))
	)
	; stop criterial
	(
		(> i Num)
	)
 
	(sdedr:define-constant-profile (string-append "DF.Dop" (number->string i))
		"PhosphorusActiveConcentration" (* i 1e15))
	(sdedr:define-constant-profile-region (string-append "PL.Dop" (number->string i))
		(string-append "DF.Dop" (number->string i)) (string-append "R.Silicon" (number->string i)) )
 
); do 

```

### **SDE批量移动代码:**

1. 选中需移动文本
2. 按住左键+TAB键，即可实现右移
3. 按住左键+shift+TAB键，即可实现左移

### **涨落的影响:**

![涨落的影响](/my_note/图片/2.4/涨落的影响.png)

### **随机数:**

随机数：掷骰子 （不可重复性）
伪随机数是用确定性的算法计算出来的随机数序列。并不真正的随机，但具有类似于随机数的统计特征，如均匀性、独立性等。在计算伪随机数时，若使用的初值(种子)不变，那么伪随机数的数序也不变。（可重复性）

### **产生随机数:**

![伪随机数](/my_note/图片/2.4/伪随机数.png)
![伪随机数高斯分布](/my_note/图片/2.4/伪随机数高斯分布.png)

```scheme{.line-numbers}

(define seed 1.0)

;defination of function 

(define (c-rand) 
	(set! seed (remainder (+ (* 110351524533 seed) 1234567) 2147483648)); mode 
	(quotient seed 65536) ;
); 0~32769 integer

; uniform numbers in open interval (0, 1)
(define (unif-rand) 
	(/ (+ (c-rand) 1) 32769.0) ;32769 = 2147483648/65536
)

; Gaussian distribution by Box-Muller method
(define (normal-rand mid sigma)
	(+ mid (* sigma (* -2.0 (log (unif-rand))) (cos (* 2 PI (unif-rand)))))
) 

; function to get a list of 'n random numbers from generator 'r '() empty list  
(define (rand-list r n) = (if (zero? n) '() (cons (r) (rand-list r (- n 1)))))
;cons连接两个列表
 
; (rand-list unif-rand 3)
; (rand-list unif-rand 3)
; ( (unif-rand) (rand-list unif-rand 2))
; ( (unif-rand) (unif-rand) (rand-list unif-rand 1))
; ( (unif-rand) (unif-rand) (unif-rand) (rand-list unif-rand 0))
; ( (unif-rand) (unif-rand) (unif-rand) '())

(define (Lgate-normal) (normal-rand 20e-3 1e-3) )
(display (Lgate-normal))
(display (Lgate-normal))

(define RAND_LIST (rand-list Lgate-normal (+ @node:index@ 1)) )

(define rand_lgate (list-ref RAND_LIST @node:index@))

(sde:ft_scalar (string-append "lgate " (number->string (* 1e3 rand_lgate) ) ))
(sde:ft_scalar "lgate  100")
在SDE右边打印100

(sdegeo:create-rectangle (position 0 0 0 ) (position rand_lgate rand_lgate 0.0 ) "Silicon" "R.channel")

(sde:save-model "n@node@")

```

### **取行号:**

```scheme{.line-numbers}

@node:index@)
; :表示竖向的；|表示横向的

```

date：20220717

# 2-5-网格划分的基础知识与评价标准

### **delaunay三角网/三角化:**

满足所有内角和最大的优化准则为delaunay三角网。这种剖分方法遵循“最小角最大”和“空外接圆”准则，两者是等效的。

空外接圆准则表示当前三角形所形成的外接圆不包含其他的点。

![1658330363628](image/2-5-网格划分的基础知识与评价标准/1658330363628.png)

![1658330608004](image/2-5-网格划分的基础知识与评价标准/1658330608004.png)

![1658330809780](image/2-5-网格划分的基础知识与评价标准/1658330809780.png)

![1658330907610](image/2-5-网格划分的基础知识与评价标准/1658330907610.png)

### **网格质量评价方法cmd输入，可以用C-shell部件来输入运行命令行的命令:**

```scheme{.line-numbers}
boxmethod n4_msh.tdr

```

![1658409069120](image/2-5-网格划分的基础知识与评价标准/1658409069120.png)

date：20220721

# 3-1-MOSFET器件静态与瞬态特性的求解

![1658557208387](image/3-1-MOSFET器件静态与瞬态特性的求解/1658557208387.png)

### **sdevice例子代码:**

```scheme{.line-numbers}

#if [string match "@type@" "NMOS"]
	#define _Vds_ 		0.1
	#define _VgsStart_ 	-0.5
	#define _VgsEnd_ 	1.0
; _VgsEnd_   加下划线是为了区分sde和sdevice，调用时也要加上下划线。单纯的字符串替换 
#else
	#define _Vds_ 		-0.1
	#define _VgsStart_ 	0.5
	#define _VgsEnd_ 	-1.0
 
#endif


* 1 input output file 
File{
	* input files 
	* Grid = "n@node|MOSFET@_msh.tdr"  
	Grid = "@tdr@"
	* Parameter = "@parameter@" * not exists

	* output files 
	Output = "@log@"
	Plot   = "@tdrdat@"
	Current= "@plot@" * n@node@_des.plt
	* other files...

}

* 2 Boundary Condition 
Electrode{
	{Name= "source" 	Voltage= 0.0 }
	{Name= "gate" 		Voltage= 0.0 }
	{Name= "drain" 		Voltage= 0.0 }
	{Name= "substrate" 	Voltage= 0.0 }
}
*Thermode{
*...
*}


* 3 Physics

Physics{ *global model
	Fermi
}
* material-wise
Physics(Material="Silicon"){
	* Drift-Diffuse Model
 
	Mobility (
	  DopingDep
	  Enormal
	  HighFieldSaturation
	)
	Recombination(
		SRH(ElectricField)  
		Auger
	)
	EffectiveIntrinsicDensity(OldSlotboom)
}

#if 0 
	* region-wise
	Physics(Region="R.channel"){
		......
	}

	*interface-wise
	Physics(MaterialInterface="Silicon/Oxide") {
		* Traps
	}
	Physics(RegionInterface="R.channel/R.gateoxide") {
		* Traps 
	}
#endif 

* 4 math 
Math{
	Method = Super
}


* 5 output
Plot{

}

CurrentPlot{
	ElectrostaticPotential((0 1e-3))
	eDensity((0 1e-3))
	srhRecombination(
		(0 1e-3)
		Integrate(Semiconductor)
	)
	eMobility(
		minimum(Window[(@<Lgate/3.0>@ 0.0) (@<-1.0*Lgate/3.0>@ 2e-3) ])
	)
}

* 6 Solve
Solve{
	#------------"pure mathmatic problem"-------#
	# Ax=b(gate= 0 drain=0) ---> tdr
	* step 1 initial solution
	*  solve poisson equation only 
	Poisson
	*  solve poisson electron hole together
	Coupled{Poisson Electron  } 
	Coupled{Poisson Electron Hole } 
	* - get a solution 
	* step 2 change boundary condition
	Quasistationary( 
	 maxstep= 0.10 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= 	_VgsStart_}
	Goal{Name= "drain" Voltage=  	_Vds_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}
	# Ax=b(gate= -0.5 drain=0.1)

	* step 3 sweep the Id-Vg curve
	* n@node@_des.plt
	NewCurrentPrefix = "IdVg_" * IdVg_n@node@_des.plt
	Quasistationary( 
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= _VgsEnd_}
	* t(time) t=0 (gate=-0.5 drain=0.1) t=1 (gate= 1.0 drain =0.1)
	){ Coupled{ Poisson Electron Hole }
		CurrentPlot(Time=(
			*time list
			Range=(0.0 1.0) Intervals= 20; *---> 0.05;0.10;0.15....1.00
			Range=(0.5 0.6) Intervals=20   *---> 0.505;0.51;0.515....0.60
			)
		)
		Plot(FilePrefix="Profile_n@node@" Time=(Range=(0 1.0) Intervals= 10 ) noOverwrite ) * save tdr files
	}

	#Goal->Current/Charge
	#Goal->Parameter

}

```

### **定义物理模型：**

```scheme{.line-numbers}


#根据材料定义
Physics(Material="Silicon"){   }

#根据区域定义
Physics(Region="R.CHANNEL"){   }

#根据材料定义界面物理模型
Physics(MaterialInterface="Silicon/0xide"){ }

#根据区域定义界面物理模型
Physics(RegionInterface="R1/R2"){   }

#重复定义的话，区域具有更高的优先级
```

![1658559631883](image/3-1-MOSFET器件静态与瞬态特性的求解/1658559631883.png)

### **sde和sdevice中代码移动：**

按住鼠标左键加TAB或shift+TAB即可实现，选中代码的右移和左移

### **选取显示在曲线中的点（控制存取点的个数）：**

按住鼠标左键加TAB或shift+TAB即可实现，选中代码的右移和左移

```scheme{.line-numbers}
CurrentPlot(Time=(
			*time list
			Range=(0.0 1.0) Intervals= 20; *---> 0.05;0.10;0.15....1.00
			Range=(0.5 0.6) Intervals=20   *---> 0.505;0.51;0.515....0.60
			)
		)

```

### 保存TDR文件save tdr files（正常求取的tdr为最后时刻求解的）：

```scheme{.line-numbers}
Plot(FilePrefix="Profile_n@node@" Time=(Range=(0 1.0) Intervals= 10 ) noOverwrite )
#noOverwrite 表示不覆盖之前保存的图片，另起一个名
```

### SDEVICE中定义变量和判断语句：

```scheme{.line-numbers}

#if [string match "@type@" "NMOS"]
	#define _Vds_ 		0.1
	#define _VgsStart_ 	-0.5
	#define _VgsEnd_ 	1.0
; _VgsEnd_   加下划线是为了区分sde和sdevice，调用时也要加上下划线。单纯的字符串替换 
#else
	#define _Vds_ 		-0.1
	#define _VgsStart_ 	0.5
	#define _VgsEnd_ 	-1.0
#endif


```

### 保存一个状态的TDR，之后用Load调用，计算

```scheme{.line-numbers}
Solve{
	#------------"pure mathmatic problem"-------#
	# Ax=b(gate= 0 drain=0 source=0 substrate=0) ---> tdr
	* step 1 initial solution
	*  solve poisson equation only 
	Poisson
	*  solve poisson electron hole together
	Coupled{Poisson Electron  } 
	Coupled{Poisson Electron Hole } 
	* - get a solution 
	* step 2 change boundary condition
	Quasistationary( 
	 maxstep= 0.10 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= _Vgs1_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}

	************method 2***********
	Save(FilePrefix= "Vg1_n@node@")

	Quasistationary( 
	 maxstep= 0.10 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= _Vgs2_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}
	Save(FilePrefix= "Vg2_n@node@")

	Quasistationary(
	 maxstep= 0.10 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= _Vgs3_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}
	Save(FilePrefix= "Vg3_n@node@")

	**IdVd curve 1 
	Load(FilePrefix= "Vg1_n@node@")
	NewCurrentFile= "IdVd_Vg1_"
	Quasistationary(
	 DoZero
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= _VdsEnd_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}

	Load(FilePrefix= "Vg2_n@node@")
	NewCurrentFile= "IdVd_Vg2_"
	Quasistationary(
	 DoZero
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= _VdsEnd_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}

	Load(FilePrefix= "Vg3_n@node@")
	NewCurrentFile= "IdVd_Vg3_"
	##----converge solution (tdr)
	* set(traps ....)
	##----unconverge solution (tdr)
	Quasistationary(
	 * t=0 converge!
	 DoZero
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= _VdsEnd_}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}




	* step 3 IdVd Vgate=0.5V
	#if 0
	********************* method 1************
	NewCurrentFile= "IdVd_Vg1_"
	Quasistationary( 
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= 1.0}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}
	* Ax=b(gate= 0.5 drain=1.0 source=0 substrate=0) ---> tdr
	NewCurrentFrefix= "tmp_"
	Quasistationary( 
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= 0.0}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}

	Quasistationary( 
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "gate"  Voltage= 0.75}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}

	NewCurrentFile= "IdVd_Vg2_"
	Quasistationary( 
	 maxstep= 0.02 minstep= 1e-5
	 increment= 1.4 decrement= 2.0
	 initialstep= 1e-3 * unit : t(time)
	Goal{Name= "drain"  Voltage= 1.0}
	* t: t=0 (gate=0 drain=0) t=1(gate=-0.5 drain=0.1)
	){ Coupled{ Poisson Electron Hole }}
	******************method 1 end************
	#endif 
	 
}



```

### Dozero：算一下零点

```scheme{.line-numbers}
两个准静态之间，默认收敛，0这个点不解。加上Dozero，求解0点

传递过程中可能发生不收敛，在中间加上Dozero，保证收敛

```

### 瞬态求解，变化过程中转折点处求解加密方法：

1.指定转折点附近的步长（适合变化陡峭）

```scheme{.line-numbers}

  NewCurrentFile="Trans_"
  Transient (
	InitialTime= 0.0 FinalTime= 4e-10
	InitialStep= 1e-12 Increment= 1.41
	Minstep= 1e-15 MaxStep= 1e-11
	TurningPoints(
	 	(Condition(Time (1e-11;0.9e-10;1.0e-10;2.0e-10;2.1e-10;2.9e-10;3.0e-10)) Value = 1e-13) * value= step
	)
  ){ Coupled { Poisson Electron Hole}  }
   ;制定转折点附近步长

```

2.运用currentplot加密某一部分求解（适合变化较缓，高斯分布）

```scheme{.line-numbers}

  NewCurrentFile="Trans_"
  Transient (
	InitialTime= 0.0 FinalTime= 4e-10
	InitialStep= 1e-12 Increment= 1.41
	Minstep= 1e-15 MaxStep= 1e-11
	#TurningPoints(
	# 	(Condition(Time (1e-11;0.9e-10;1.0e-10;2.0e-10;2.1e-10;2.9e-10;3.0e-10)) Value = 1e-13) * value= step
	#)
  ){ Coupled { Poisson Electron Hole}  
   	CurrentPlot( Time=(
   		# global 全局
		Range=(0.0e-10  4.0e-10) 	 Intervals= 50;
		# swiching time zone 发生切换的区间
		Range=(0.9e-10  1.1e-10) 	 Intervals= 30;
		Range=(2.0e-10  2.2e-10) 	 Intervals= 30 
	) )
  }
 
}


```

### Currentplot:观察某点处的某一物理量变化趋势

```scheme{.line-numbers}
CurrentPlot{
	ElectrostaticPotential((0 1e-3))
	eDensity((0 1e-3))
	srhRecombination(
		(0 1e-3)
		Integrate(Semiconductor)
	)
	eMobility(
		minimum(Window[(@<Lgate/3.0>@ 0.0) (@<-1.0*Lgate/3.0>@ 2e-3) ])
	)
}

```

### 可以扫描的量

Goal Current/charge

Goal Parameter

```scheme{.line-numbers}
CurrentPlot{
	Model="ConstantMobility_electron"  Parameter=mumax"
}
NewCurrentFile="Trans_"
  Transient (
	InitialTime= 0.0 FinalTime= 4e-10
	InitialStep= 1e-12 Increment= 1.41
	Minstep= 1e-15 MaxStep= 1e-11

#Material=Silicon, Model=ConstantMoblllty_electron,Parameter=mumax,Value=1417

Goal {Model="ConstantMobility_electron"  Parameter=mumax"  Value=500}

  #  Goal{ Model= 	Parameter= 	Value=        }
	#TurningPoints(
	# 	(Condition(Time (1e-11;0.9e-10;1.0e-10;2.0e-10;2.1e-10;2.9e-10;3.0e-10)) Value = 1e-13) * value= step
	#)
  ){ Coupled { Poisson Electron Hole}
    CurrentPlot( Time=(
   		# global 全局
		Range=(0.0e-10  4.0e-10) 	 Intervals= 50;
		# swiching time zone 发生切换的区间
		Range=(0.9e-10  1.1e-10) 	 Intervals= 30;
		Range=(2.0e-10  2.2e-10) 	 Intervals= 30
	) )
  }

}

```

date:20220725

# 3-2.TCL编程与SVISUAL自动化数据处理（一）

### **准静态：经过足够长的时间，器件内部的所有参数都不再变化**

### **SVisual代码例子：**

```scheme{.line-numbers}
设置运行依赖，前面节点未运行，这个节点就不能运行
#setdep  @previous@


# dataset 加载数据
load_file ./IdVg_n@node|IdVg@_des.plt -name  dataset_IdVg_n@node@
# plot  画图
if { [lsearch [list_plots] plot_IdVg ]== -1} {
	create_plot -1d -name plot_IdVg
}

if [info exists runVisualizerNodesTogether] {
	set legend_text_curve $legend
	set common_text_curve $commonParameters
}  else  {
	set legend_text_curve "@type@"
	set common_text_curve "@type@"
}


选中数据
select_plots {plot_IdVg}

# create absIds variable 创建曲线
create_variable -name absIds -dataset dataset_IdVg_n@node@ -function {abs(<drain TotalCurrent:dataset_IdVg_n@node@>)}
# create a curve 
create_curve -plot plot_IdVg -dataset {dataset_IdVg_n@node@} -axisX {gate OuterVoltage} -axisY {absIds} -name curve_IdVg_n@node@

# compute Gm  计算跨导
create_variable -name Gm -dataset dataset_IdVg_n@node@ -function {abs(diff(<drain TotalCurrent:dataset_IdVg_n@node@>,<gate OuterVoltage:dataset_IdVg_n@node@>))}
create_curve -plot plot_IdVg -dataset {dataset_IdVg_n@node@} -axisX {gate OuterVoltage} -axisY2 {Gm} -name curve_Gm_n@node@


set_curve_prop -plot plot_IdVg {curve_IdVg_n@node@} -label "I<sub>DS</sub>V<sub>GS</sub>:$legend_text_curve"
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -label "G<sub>m</sub>:$legend_text_curve"
set_axis_prop -plot plot_IdVg -axis y -type log



# line width 线宽
set_curve_prop -plot plot_IdVg {curve_IdVg_n@node@} -line_width 3
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -line_width 3 
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -line_style dash

# color 颜色
set COLORS  [list green blue red orange magenta violet brown #008000 ] 
set i_experiment @node:index@
set Color_Length [llength $COLORS]
set color_now [lindex $COLORS [expr $i_experiment%$Color_Length]] 

多条曲线的颜色变化
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@ curve_IdVg_n@node@} -color $color_now
 

# axis - text 
set_axis_prop -plot plot_IdVg -axis x -title "Gate Voltage, V<sub>GS</sub> \[V\]"
set_axis_prop -plot plot_IdVg -axis y -title "Drain Current, I<sub>DS</sub> \[A/<greek>m</greek>m\]"
set_axis_prop -plot plot_IdVg -axis y2 -title "Transconductance, G<sub>m</sub> \[S\]"


# title text 
set_plot_prop -plot {plot_IdVg} -title " I<sub>DS</sub>V<sub>GS</sub> Curve $common_text_curve"
set_axis_prop -plot plot_IdVg -axis x -title_font_family arial -title_font_size 14 -title_font_color #000000 -title_font_att normal
set_axis_prop -plot plot_IdVg -axis y -title_font_family arial -title_font_size 14 -title_font_color #000000 -title_font_att normal
set_axis_prop -plot plot_IdVg -axis y2 -title_font_family arial -title_font_size 14 -title_font_color #000000 -title_font_att normal

set_axis_prop -plot plot_IdVg -axis x -scale_font_family arial -scale_font_size 12 -scale_font_color #000000 -scale_font_att normal
set_axis_prop -plot plot_IdVg -axis y -scale_font_family arial -scale_font_size 12 -scale_font_color #000000 -scale_font_att normal
set_axis_prop -plot plot_IdVg -axis y2 -scale_font_family arial -scale_font_size 12 -scale_font_color #000000 -scale_font_att normal

set_plot_prop -plot {plot_IdVg} -title_font_family arial -title_font_size 13 -title_font_color #000000 -title_font_att normal


load_library extract

set VDATA [get_curve_data curve_IdVg_n@node@ -axisX]
set IDATA [get_curve_data curve_IdVg_n@node@ -axisY]
puts "VDATA:"
puts $VDATA
puts "IDATA:"
puts $IDATA
# 100nA/um (areafactor =1)
set I0 100e-9
set Itmp 200e-9 

ext::ExtractVti -out Vti_Lin -name "Vti_Lin" -v $VDATA -i $IDATA -io $I0
ext::ExtractVti -out Vtmp -name "noprint" -v $VDATA -i $IDATA -io $Itmp
ext::ExtractExtremum -out Ion  -name "Ion"  -x $VDATA -y $IDATA  -type "max"  
ext::ExtractExtremum -out Ioff  -name "Ioff"  -x $VDATA -y $IDATA  -type "min"
ext::ExtractSS -out SS -name "SS" -v $VDATA -i $IDATA -vo $Vti_Lin

set OnOffRatio [expr log(abs($Ion/$Ioff))/log(10)]
puts "DOE: OnOffRatio [format %.3f $OnOffRatio ]"
# puts "DOE: Vti_Lin $Vti_Lin"


导出csv文件和图片
# export xy data to csv files
export_curves {curve_IdVg_n@node@ curve_Gm_n@node@ } -plot plot_IdVg -filename ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.csv -format csv  -overwrite
# export curve plot to png file
export_view ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.png -plots {plot_IdVg} -format png -overwrite


```

### 读命令的返回值：[]

```scheme{.line-numbers}
定义d等于a和b的值相加的值
set d [exor $a+$b]
```

### **expr命令：**

```scheme{.line-numbers}
expr 1/2
输出0

expr 1.0/2
输出0.5

注意整数和浮点数区别
```

### **常用命令：**

```scheme{.line-numbers}
set pi [expr 2.0*asin(1.0)]
#-> 3.14159265359
set Word "Hello World"
#-> Hello World
string length Word
#-> 4
string length $Word
#-> 11
append Word "!”
#-> Hello World!
string index $Word 2
#-> |
string range $Word 1 3
#-> ell
string equal "NMOS" "NMOS"
#-> 1 (表示真)



```

### **字符串比较：**

```scheme{.line-numbers}
string equal "NMOS" "NMOS"
#-> 1
string equal "NMOS" "PMOS"
#-> 0

1为真，0为假

string match "*NMOS*" "This is NMOS"
#-> 1
string match "*NMOS*" "This is nmos"
#-> 0
发现"This is NMOS"中有NMOS，就为true

string match -nocase NMOS** "This is nmos"
#->1
-nocase 忽略大小写

```

### **字符转义：**

```scheme{.line-numbers}
puts "a = $a'
a=1.000

puts "a=\$a"
a=$a

puts "Material=\"Silicon\""
Material="Silicon"

puts "[Material=\"Silicon\'"\]"
[Material="Silicon"]

puts{a=$a}
a=$a

puts {a=$a b=$b}
a=$a b=$b 

puts "{a=$a b=$b}"
{a=1.000 b=2.0}

```

### **list（列表）相关：**

```scheme{.line-numbers}
set NUMlist [list 1 2 3 4 5]
#->1 2 3 4 5
set NUMlist { 1 2 3 4 5}
#->1 2 3 4 5
set ABClist { a b c d e}
#-> a b c d e
set STRlist {this is a sentence}
#-> this is a sentence
set MIXlist{ 1 2 a b c ? { 3 4 }}
#-> 1 2 a b c ? { 3 4 }
set EMPTYlist [list]   空列表
length $ABClist    返回列表长度
#-> 5
lindex $ABClist 1
#-> b
lindex $ABClist 0
#-> a
lindex $ABClist 4
#-> e
lappend ABClist f 增加列表的值
#-> a b c d e f 
lsearch $ABClist f  返回该值在列表所在位置
#-> 5
lsearch $ABClist k   -1表示没有找到
#-> -1

```

### **循环相关：**

```scheme{.line-numbers}
# loop 

set ABClist [list A B C D E F G]
set NUMlist [list 1 2 3 4 5 6 7]

# type 1
puts "-----foreach-----"
foreach abc $ABClist num $NUMlist {
	puts "${num}_$abc"
}


# type 2 
# incr : increase : c i++
puts "-----  for  -----"
for {set i 0} { $i< [llength $ABClist] } { incr i } {
	puts "[lindex $NUMlist $i]_[lindex $ABClist $i]"
# [lindex $NUMlist $i] 取出$NUMlist中第i个值
}

# type 3
puts "-----while-----"
set i 0
while {$i<[llength $ABClist]} {
	puts "[lindex $NUMlist $i]_[lindex $ABClist $i]"
	incr i
	#break
}


# if-else 判断
puts "-----if-----"
set val 0.0 
set val2  0

值比较
if {$val==0} {
	puts "val is zero"
} else {
	puts "val:$val"
}

字符串比较 
&& 与
|| 或
if { [string equal $val $val2] && $val>0 || $val2>0 } {
	puts "val = val2"
} else {
	puts "val != val2"
}
 

#format 

set pi [expr 2.0*asin(1.0)]
puts "pi=$pi"
输出小数点后三位的浮点数
puts "pi=[format %.3f $pi]"

#output 
set node @node@

DOE: 会把数据拿出来，在swb中输出 node名称 $node值
puts "DOE: NODE $node"
puts "DOE: name lan.xiaomu"
puts "DOE: pi $pi"
puts "DOE: pi2 [format %.3f $pi]"


# function 
proc ADD { a { b 1.0 } } {
	set c [expr $a+$b]
	return $c
}
;(b 1.0) b不给的话，默认为1.0

puts [ADD 1.0 2.5]
输出3.5
puts [ADD 1.0]
输出2

# file 
[glob "*"]：获取当前路径下所有文件 *匹配所有文件
[glob "*.tdr"] 获取当前路径下所有tdr文件
用于批处理

set filelist [glob "IdVg*des.plt"]
puts "there are [llength $filelist] IdVg plt files"
foreach file $filelist {
	puts $file 
}

```

### **Svisual保存相同量和不同量：**

```scheme{.line-numbers}
if [info exists runVisualizerNodesTogether] {
	set legend_text_curve $legend
	set common_text_curve $commonParameters
}  else  {
	set legend_text_curve "@type@"
	set common_text_curve "@type@"
}
$legend 保存的不同的量
$commonParameters 保存的相同的量
```

### **判断是否存在画板：**

```scheme{.line-numbers}
if { [lsearch [list_plots] plot_IdVg ]== -1} {
	create_plot -1d -name plot_IdVg
}


```

### **颜色循环：**

```scheme{.line-numbers}
# color 颜色
set COLORS  [list green blue red orange magenta violet brown #008000 ] 设置颜色标号
set i_experiment @node:index@   获取第几列标号
set Color_Length [llength $COLORS]   获取颜色列表长度
set color_now [lindex $COLORS [expr $i_experiment%$Color_Length]]  对颜色长度取余

多条曲线的颜色变化
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@ curve_IdVg_n@node@} -color $color_now

```

### **导出数据：**

```scheme{.line-numbers}
导出csv文件和图片 必须加overwrite，否则只会保存第一次运行的文件，保存图像需要用batchx模式
# export xy data to csv files
export_curves {curve_IdVg_n@node@ curve_Gm_n@node@ } -plot plot_IdVg -filename ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.csv -format csv  -overwrite
# export curve plot to png file
export_view ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.png -plots {plot_IdVg} -format png -overwrite


```

### **linux命令：**

```scheme{.line-numbers}
mkdir exported_data   在当前路径下创建exported_data文件夹
cd exported_data/     cd 改变工作目录
ls   展现当前文件夹下所有文件
nautilus ./   打开文件管理器
```

### **设置依赖，sdevice也可以添加：**

```scheme{.line-numbers}
设置运行依赖，前面节点未运行，这个节点就不能运行
#setdep  @previous@

```

date：20220726

# 3.3-器件仿真中的物理模型（一）

### **svisual 中换行：**

```scheme{.line-numbers}
运用 \ 换行
create curve -plotplot IV -dataset {dataset_IV3_ndnoded) -axisx {drain_Outervoltage} \
axisY {drain_TotalCurrent} -name curve_IV3_n@node@
```

![1658979767799](image/3.3-器件仿真中的物理模型（一）/1658979767799.png)

![1658979947211](image/3.3-器件仿真中的物理模型（一）/1658979947211.png)

### **单载流子仿真：**

```scheme{.line-numbers}
仿真MOSFET时，电子起主导作用，耦合求解电子和泊松与（电子、泊松、空穴）相差不大，因此求解（电子和泊松）会有更好的求解速度和收敛性，但空穴的准费米势是常数
```

### **反向倒推空穴准费米势：**

```scheme{.line-numbers}
RecomputeQFP after the solution of Coupled Poisson Electron update Efp
```

### **雪崩击穿：**

```scheme{.line-numbers}
不能用单载流子仿真，会有大的误差
```

### **Dopingwell：**

*.plt文件中Electrode Charge的意义
每个DopingWel中的准费米势可以被控制和扫描

```scheme{.line-numbers}
Plot{
DopingWells
}
```

### **SVisual代码例子：**

![1658981402165](image/3.3-器件仿真中的物理模型（一）/1658981402165.png)

![1658981592448](image/3.3-器件仿真中的物理模型（一）/1658981592448.png)![1658981676127](image/3.3-器件仿真中的物理模型（一）/1658981676127.png)![1658981757221](image/3.3-器件仿真中的物理模型（一）/1658981757221.png)![1658982211494](image/3.3-器件仿真中的物理模型（一）/1658982211494.png)![1658982623084](image/3.3-器件仿真中的物理模型（一）/1658982623084.png)

### **肖特基势垒：**

### **钉扎效应：**

### **势垒降低效应（镜像力）：**

### **阻性接触：**

![1658983812782](image/3.3-器件仿真中的物理模型（一）/1658983812782.png)![1658984305015](image/3.3-器件仿真中的物理模型（一）/1658984305015.png)![1658984391487](image/3.3-器件仿真中的物理模型（一）/1658984391487.png)

### 参考资料：

[DAY10 仿真中的物理模型概述（一）-2021-10-25](F:/%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%BA%90/%E5%B7%A5%E5%85%B7%E7%94%B5%E5%AD%90%E4%B9%A6/sentaurus%E5%BB%BA%E6%A8%A1%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%96%99/%E8%B5%84%E6%96%99/pdf%E8%B5%84%E6%96%99/DAY10%20%E4%BB%BF%E7%9C%9F%E4%B8%AD%E7%9A%84%E7%89%A9%E7%90%86%E6%A8%A1%E5%9E%8B%E6%A6%82%E8%BF%B0%EF%BC%88%E4%B8%80%EF%BC%89-2021-10-25.pdf "提示词")

date：20220728

# 3-4-物理模型（二）与温度仿真

### **主要内容：产生复合模型、SRH、俄歇复合、温度仿真、热边界**

### **产生复合模型分类：**

![1662097304598](image/3-4-物理模型（二）与温度仿真/1662097304598.png)

### **SRH复合：**

![1662097919639](image/3-4-物理模型（二）与温度仿真/1662097919639.png)

### **恒定载流子产生：**

![1662364063883](image/3-4-物理模型（二）与温度仿真/1662364063883.png)

### **热特性即边界条件：**

![1662364153085](image/3-4-物理模型（二）与温度仿真/1662364153085.png)![1662364225036](image/3-4-物理模型（二）与温度仿真/1662364225036.png)![1662364237279](image/3-4-物理模型（二）与温度仿真/1662364237279.png)![1662364261158](image/3-4-物理模型（二）与温度仿真/1662364261158.png)

### **温度方程：**

![1662364340120](image/3-4-物理模型（二）与温度仿真/1662364340120.png)![1662364387129](image/3-4-物理模型（二）与温度仿真/1662364387129.png)![1662364412457](image/3-4-物理模型（二）与温度仿真/1662364412457.png)

### **热力学模型：**

![1662364703330](image/3-4-物理模型（二）与温度仿真/1662364703330.png)![1662364728244](image/3-4-物理模型（二）与温度仿真/1662364728244.png)

date：20220905

# 3-5-碰撞电离与击穿特性仿真物理模型（二）与温度仿真

### **Areafactor：出现的areafactor会相乘，下面图中的面积为1e12，resist和areafactor相关，会除以areafactor。resistor是实际电阻，和areafactor无关。**

![1662452236507](image/3-5-碰撞电离与击穿特性仿真/1662452236507.png)

### **碰撞电离：**

![1662452861647](image/3-5-碰撞电离与击穿特性仿真/1662452861647.png)

AvalDensGradQF(不收敛时可以加上)

忽略扁平单元（一般不超过2度，忽略小于2度的偏平单元）

![1662453052144](image/3-5-碰撞电离与击穿特性仿真/1662453052144.png)![1662453392911](image/3-5-碰撞电离与击穿特性仿真/1662453392911.png)![1662453492498](image/3-5-碰撞电离与击穿特性仿真/1662453492498.png) ![1662454027914](image/3-5-碰撞电离与击穿特性仿真/1662454027914.png)![1662454114234](image/3-5-碰撞电离与击穿特性仿真/1662454114234.png)![1662454212294](image/3-5-碰撞电离与击穿特性仿真/1662454212294.png) ![1662454588295](image/3-5-碰撞电离与击穿特性仿真/1662454588295.png)![1662455507996](image/3-5-碰撞电离与击穿特性仿真/1662455507996.png)

### **Voltage to Current（准静态求解）：**

* 两种模式之间衔接需要一个较小的步长来保证过渡
* 下一个求解模式要注意衔接步长
* 可以求解负阻效应的击穿曲线
* 缺点：需要先求解一次，确定电压电流模式转换点（重点），不适合批量求解

![1662533191794](image/3-5-碰撞电离与击穿特性仿真/1662533191794.png)

### **串电阻法：**

可以解决负阻效应（二次击穿）和击穿处的小步长问题，不需要知道转折点的信息

resistor = 1e11

voltage = 1e7       #   I = V/R =1e-4

![1662533950215](image/3-5-碰撞电离与击穿特性仿真/1662533950215.png)

### Continuation：Curve Trace：

相当于加一个动态的负载电阻

可能会返回求解

求解步长比较均匀

![1662535049698](image/3-5-碰撞电离与击穿特性仿真/1662535049698.png)

![1662534713493](image/3-5-碰撞电离与击穿特性仿真/1662534713493.png)

![1662534856708](image/3-5-碰撞电离与击穿特性仿真/1662534856708.png)

### 瞬态求解：

宽禁带半导体用准静态求解比较困难，用瞬态求解来改善收敛性

瞬态时间足够长，可以认为是准静态

时间取的值要稍微大一点，时间不一致会存在差异，时间够长和准静态相差不大

date：20220907

# 3-6-SPICE模型、混合模式与sdevice编程：一个环振的仿真

![1662541148644](image/3-6-SPICE模型、混合模式与sdevice编程：一个环振的仿真/1662541148644.png)![1662541226997](image/3-6-SPICE模型、混合模式与sdevice编程：一个环振的仿真/1662541226997.png)

![1662541327694](image/3-6-SPICE模型、混合模式与sdevice编程：一个环振的仿真/1662541327694.png)

### Spice 模型

![1662541410433](image/3-6-SPICE模型、混合模式与sdevice编程：一个环振的仿真/1662541410433.png)

### 例子：

```scheme{.line-numbers}
System {
	* SPICE model/TCAD model + Netlist
	* choice1 : HSPICE netlist (.sp)
	* choice2 : SPICE circuit file (.scf)
	* by command

	NMOS nmos (source=s drain=d gate=g substrate=b )
	# power supply
	Vsource_pset Vg (g 0) {dc = 0.0} #instance parameter 
	Vsource_pset Vs (s 0) {dc = 0.0} #instance parameter 
	Vsource_pset Vd (d 0) {dc = 0.0} #instance parameter 
	Vsource_pset Vb (b 0) {dc = 0.0} #instance parameter 
	#save the result at the system nodes
	Plot "n@node@_sys_des.plt" (time() v(g) v(s) v(d) v(b)
									i(nmos,s) i(nmos d) i(nmos g) i(nmos b)
								)

}


Solve{

   #step 1:  inital solution 
   Poisson #without (contact equation) 
   Coupled{ Poisson Electron   }
   Coupled{ Poisson Electron   Contact Circuit }
   
   
   #step 2: prepare
  Quasistationary ( 
    InitialStep= 1e-2 Increment= 1.41
    MinStep= 1e-7     MaxStep= 0.2
    Goal { Parameter= Vd.dc Value= 1.5 }
  ){
    Coupled { 
      Poisson Electron   Contact Circuit
    }
  }
  #step 3: simulate IdVg Curve #
  NewCurrentFile= "IdVg_"
  Quasistationary ( 
    InitialStep= 1e-2 Increment= 1.41
    MinStep= 1e-7     MaxStep= 0.2
    Goal { Parameter= Vg.dc Value= 1.5 }
  ){
    Coupled { 
      Poisson Electron   Contact Circuit
    }
  }
  
}

```

### 1.

```scheme{.line-numbers}
Poisson nmos.Electron pmos.Hole Contact Circuit # 求解某个实例的电子、空穴方程

Grid= "n@node|MOSFET:+1@_msh.tdr" # :-> Same Col Different Row |-> Same Row different Col
# 通过：和|寻找某一行，某一列的节点号

右键 configuration -> prune 屏蔽某个节点，不运行


NMOS nmos (source=ss drain=out gate=in substrate=ss ) {Physics{Areafactor=2.0}} #w=1.0um 
PMOS pmos (source=dd drain=out gate=in substrate=ss ) {Physics{Areafactor=10.0}} #w=1.0um
#可以在实例后面补充物理参数和其他参数
```

### 2.在Sdevice中用tcl代码块实现循环

```scheme{.line-numbers}
!(
		for {set i 1} {$i<=$stage} {incr i} {
			if { $i == 1} {
				puts "NMOS nmos$i (source=ss drain=out$i gate=out$stage substrate=ss ) \{Physics\{Areafactor=1.0\}\}"
				puts "PMOS pmos$i (source=dd drain=out$i gate=out$stage substrate=ss ) \{Physics\{Areafactor=1.0\}\}" 
			} else {
				puts "NMOS nmos$i (source=ss drain=out$i gate=out[expr $i-1] substrate=ss ) \{Physics\{Areafactor=1.0\}\}"
				puts "PMOS pmos$i (source=dd drain=out$i gate=out[expr $i-1] substrate=ss ) \{Physics\{Areafactor=1.0\}\}" 
			}
		}

	)!
#其中{}[]等特殊符号需要加\进行转义
```

### 3.Svisual中匹配相对应代码

```scheme{.line-numbers}

set stage @stage@
for {set i 1} {$i<= $stage} {incr i} {
	create_curve -plot plot_RO -dataset {dataset_RO_n@node@} -axisX {time} -axisY "{v(out$i)}" -name curve_${i}_n@node@
	set_curve_prop -plot plot_RO "{curve_${i}_n@node@}"   -label "V<sub>OUT$i</sub> $legend_text_curve"
	# line width 
	set_curve_prop -plot plot_RO "{curve_${i}_n@node@}" -line_width 3

}
# 由于{}的存在，必须加""才能保证i的转义 ""{curve_${i}_n@node@}" 、
#${i} 通过加{}保证要转义的字符部分

```

date：20220908

# 3-7：混合模式（二）：修改SPICE模型参数 - SVISUAL动画自动生成

### 主要内容：SPICE模型和TCAD模型的对应，以及修改SPICE模型中的默认参数及调用。

### 电压源的pulse设置方法

```scheme{.line-numbers}
Vsource_pset Vin 	(in 0) 	{ pulse= (   0       2.0  @<2*time_ps>@  @<0.2*time_ps>@  @<0.2*time_ps>@    @<12*time_ps>@   @<24.4*time_ps>@)
															#equal to : [expr 0.2*$time_ps]
	} 

```

### 插入文件代码

```scheme{.line-numbers}
Insert= "PlotSection_des.cmd" # sdevice command  : exec at sdevice   @var@ is not valid!
#include "MathSection_des.cmd" 
   # #include----> swb preprocess command: exec at preprocess  @var@

```

### gif图实现例子

```scheme{.line-numbers}
#setdep @previous@

# step 1 get the file list 

#lsort -ascii -increasing 按ascii码递增排序   -type f 找到类型是文件的东西  glob找东西  *通配符，表示这个位置是任意值
set FileList [lsort -ascii -increasing [glob -type f "profile_n@node|Transient@_*_IGBT_des.tdr"]]
puts "Find the TDR files:-----------"
foreach file $FileList { puts "$file" }




# step 2 start movie
set i 1

start_movie 

set FrameList [list]

set tStart 0
set tFinal 3e-6

set NumOfFiles [llength $FileList]

set dt [expr 1.0*($tFinal-$tStart)/($NumOfFiles-1)]

foreach file $FileList {
	puts -nonewline "processing file:  $file ..."
	#load file
	load_file $file -name dataset_tdr_$i
	create_plot -dataset dataset_tdr_$i -name plot_tdr_$i
	select_plots "{plot_tdr_$i}" 
 
	# add frame
	#- prepare my device 			# geom = dataset 
	set_field_prop -plot plot_tdr_$i -geom dataset_tdr_$i @Field@ -show_bands
	set_field_prop -plot plot_tdr_$i -geom dataset_tdr_$i @Field@ -max @maxval@ -max_fixed
	set_field_prop -plot plot_tdr_$i -geom dataset_tdr_$i @Field@ -min @minval@ -min_fixed
	set t0 [expr ($i-1)*$dt+$tStart]
	if { $t0<1.2e-6 && $t0>2e-7 } {
		set OnOffStatus "ON"
	} else {
		set OnOffStatus "OFF"
	}
	set_plot_prop -plot "{plot_tdr_$i}" -title "t= [format %.2f [expr $t0*1e6]]us ($OnOffStatus)"
	zoom_plot -plot plot_tdr_$i -window {8.14806 13.9461 -1.98341 -2.18156}
	set_material_prop {DepletionRegion} -plot plot_tdr_$i -geom dataset_tdr_$i -off
	set_field_prop -plot plot_tdr_$i -geom dataset_tdr_$i LatticeTemperature -levels 99


	#- add frame 
	add_frame -plot plot_tdr_$i -name Frame$i
	lappend FrameList Frame$i

	#unload file
	remove_plots plot_tdr_$i
	remove_datasets dataset_tdr_$i
	# next i
	incr i
	puts "done!"
}

export_movie -filename ./my_gifs/n@node@_@Field@.gif -frames $FrameList -frame_duration 5 -overwrite
stop_movie
```

### 求取某个场的最大值

```scheme{.line-numbers}
#建立依赖，前面运行完，才能运行这个
#setdep @node|Transient@
#set maxval x
#set minval x
#这样预处理就不会报错，#set，先定义一个值，后面替换，如果这个变量已经存在就不会替换。（提前声明1）
# step 1 get the file list 
set FileList [lsort -ascii -increasing [glob -type f "profile_n@node|Transient@_*_IGBT_des.tdr"]]
puts "Find the TDR files:-----------"
foreach file $FileList { puts "$file" }
 
# step 2 start movie
set i 1

set maxval -1e200 
set minval 1e200

foreach file $FileList {
	puts -nonewline "processing file:  $file ..."
	#load file
	load_file $file -name dataset_tdr_$i
	create_plot -dataset dataset_tdr_$i -name plot_tdr_$i
	select_plots "{plot_tdr_$i}" 
	#find max/min value 
	set min_value_in_tdr [lindex [calculate_field_value -plot plot_tdr_$i -geom dataset_tdr_$i -field @Field@ -min] 0]
#取得列表里第一个元素 [lindex 列表名 0]
	set max_value_in_tdr [lindex [calculate_field_value -plot plot_tdr_$i -geom dataset_tdr_$i -field @Field@ -max] 0]
 	if { $maxval<$max_value_in_tdr } { set maxval $max_value_in_tdr }
	if { $minval>$min_value_in_tdr } { set minval $min_value_in_tdr }
	#unload file
	remove_plots plot_tdr_$i
	remove_datasets dataset_tdr_$i
	# next i
	incr i
	puts "done!"
}

puts "DOE: maxval $maxval"
puts "DOE: minval $minval"
#输出给SWB，可以使用@minval@进行调用。

```

### 隐藏坐标轴

```scheme{.line-numbers}
set_plot_prop -plot {Plot_n1654_des} -hide_axes
```

date：20220913

# 3-8：缺陷与辐照特性仿真（单粒子效应）

### 缺陷设置代码

```scheme{.line-numbers}


#   #if ![string match "-" "@traps@"]
   ##if ![string match "*interface*" "@traps@"]
	Traps(
   	   # level exponential Gaussian table(experiment)
   	   # bulk traps (cm^-3)
   	   #if [string match "*Acceptor*" "@traps@"]
		   # acceptor exponential tail of the conduction band 
		   (Acceptor Exponential Conc=1.5e19 EnergySig=0.05 EnergyMid=0 fromCondBand 
		   hXsection= 1e-14 eXsection=1e-15 )
		   # acceptor deep levels
		   (Acceptor Gaussian Conc=1.9e18 EnergySig=0.1 EnergyMid=0.08 fromMidBandGap )
   	   #endif 
   	   #if [string match "*Donor*" "@traps@"]
		   (Donor 	Exponential 	Conc=1e18		fromValBand 	EnergyMid=0 	EnergySig=0.05)
   	   	   (Donor 	Gaussian 		Conc=1.13e18	fromValBand  	EnergyMid=0.51 	EnergySig=0.1)
   	   #endif    	   
  
   )
   ##endif
   #endif

####### EnergySig 浓度衰减系数  EnergyMid=0.08 能量参考点（峰值浓度位置）  fromcondband以导带为参考位置
#######  带尾态影响IDVG曲线后半段，深能级影响IDVG曲线中间的亚阈值区
####### Gaussian 深能级                        exponential 带尾态
####### sde "-AI" （add interface region）
####### 固定电荷 conc大于零为正固定电荷，为负为负的固定电荷。其他缺陷中conc必须大于零。
```

### 重离子入射（单粒子烧毁）代码

```scheme{.line-numbers}
	HeavyIon(
		# angel 0-180 
		Direction = (@<cos(angle/180.0*3.1415926)>@,@<sin(angle/180.0*3.1415926)>@) # (vx vy) vector 
		Location = (@x0@,0) #(x0 y0)
		Time = 1.0e-10 #0.1ns
		Length = [0.1 0.2 0.3 0.4 ]      # unit ???	[PicCoulomb um] [None: cm]
		Wt_hi  = [0.05 0.1 0.2 0.1 ]# unit ??? [PicCoulomb um] [None: cm]
		LET_f =  [0.1 0.2 0.3 0.2 ] #unit [PicCoulomb pC/um] [None: Pairs/cm3]
		Gaussian # lateral distribution
		PicoCoulomb
	)
```

### 缺陷位置随机分布代码

```scheme{.line-numbers}
Physics(Material="Silicon"){
	* Drift-Diffuse Model
 
	Mobility (
	  DopingDep
	  Enormal
	  HighFieldSaturation
	)
	Recombination(
		SRH  
		Auger
	)
	EffectiveIntrinsicDensity(OldSlotboom)

   #if ![string match "-" "@model@"]
   ##if ![string match "*interface*" "@traps@"]
	Traps(
 	   #if [string match "*SpatialShape*"  "@model@"]
   	  	 (Acceptor Exponential Conc=1.5e19 EnergySig=0.05 EnergyMid=0 fromCondBand 
   	  	 	 #if [string match "*uniform*"  "@model@"]
   	  	 	 	SpatialShape=Uniform SpaceMid= (0.0 0.01 0.00) SpaceSig= (0.01 0.01 0.01)
   	  	 	 #endif 
   	  	 	 #if [string match "*gaussian*"  "@model@"]
   	  	 	 	SpatialShape=Gaussian SpaceMid= (0.0 0.01 0.00) SpaceSig= (0.01 0.01 0.01)
   	  	 	 #endif
   	  	 )
   	   #endif
   	   
   	   #if [string match "*Rand*" "@model@"]
   	   		#define _Rand_ Randomize  
   	   #else 
   	   		#define _Rand_ 
   	   #endif
   	   
   	   #if [string match "*SingleTrap*"  "@model@"]
   	   # only: level
   	   		(Acceptor SingleTrap Level Conc= 1e22 EnergyMid=0 fromMidBandgap SpaceMid=(0.0 0.005 0.00)
   	   			_Rand_
   	   		)	   
   	   #endif
   	   
   	   #if [string match "*DeepLevels*" "@model@"]
   	   		(Acceptor SFactor= "DeepLevels" Gaussian EnergyMid=0 EnergySig= 0.05 fromMidBandGap)
   	   #endif
   	   
   	   #if [string match "*SpatialLevel*" "@model@"]
   	   		(Acceptor Level Conc= 1e22 EnergyMid=0 fromMidBandgap 
   	   			SpatialShape=Gaussian SpaceMid= (0.0 0.01 0.00) SpaceSig= (0.01 0.01 0.01)
   	   			_Rand_
   	   		)
   	   #endif
   	   
   )
   ##endif
   #endif
}

```

![1665972759041](image/3-8：缺陷与辐照特性仿真（单粒子效应）/1665972759041.png)

date：20221017

# 3-9：缺陷（二）与辐照特性仿真（总剂量效应）

### 总剂量代码设置

```scheme{.line-numbers}
# TID model
Physics{
	Radiation(
		DoseRate= @DoseRate@ # rad/s
		DoseTime= (0,@<1.0*1e6*Dose/DoseRate>@)  # TID = DoseRate*DoseTime
		# Dose= #rad/ DoseTime=(,) 
		# DoseTSigma= 
	)
}

```

### 求解设置

```scheme{.line-numbers}

	Set(TrapFilling= Empty) #不考虑缺陷

	unset(TrapFilling)  #开始考虑缺陷

	Set(TrapFilling=Frozen)  #保持缺陷的占据状态

```

### 仿真位错层错

```scheme{.line-numbers}
1.在某一位置范围内添加缺陷来代替层错、晶界
2.用随机数，来随机Si中出现缺陷
```

date：20221018

# 3-10：交流小信号分析

### 代码

```scheme{.line-numbers}

System{
	Resistor_pset r1 (in out) {resistance= 10}
	Capacitor_pset c1 (out gnd) {capacitance = 1e-9}
	# dc power 
	Vsource_pset vin (in gnd) {dc = 0.0}
	Vsource_pset vout (out gnd) {dc = 0.0}
	Vsource_pset vgnd (gnd 0 ) {dc = 0.0}
}

File{
	Output = "@log@"
	ACExtract = "n@node@"
}

Solve{

	ACCoupled(
		StartFrequency = 1e3 EndFrequency=1e3 NumberOfPoints = 1 Decade
		#
		Node( in out gnd )
		#
		Exclude(vin vout vgnd)
	){
		Circuit
	}
 
}

######  关键是 Node 和 Exclude 的设置，决定了小信号分析的正确与否
######  decade 表示频率按指数变化
```

![1666075906925](image/3-10：交流小信号分析/1666075906925.png)![1666075957626](image/3-10：交流小信号分析/1666075957626.png)

### AC绘图代码

```scheme{.line-numbers}
 
# load RF extraction Library
load_library rfx


# load file
rfx::Load -file "CV_n@node|MOSFET_AC@_ac_des.plt" -dataset dataset_ac_n@node@  \
	-port1 g -port2 d -biasport "v(g)"

#
puts "***********************************************"
puts "The Freq List:"
puts "$rfx::freq"
puts "***********************************************"
puts "The Bias List:"
puts "$rfx::bias"
puts "***********************************************"
puts "The Number of Bias condation:"
puts "$rfx::nbias"
puts "***********************************************"
puts "The Number of Freq condation:"
puts "$rfx::nfreq"
puts "***********************************************"
puts "The Start Number of Freq Index:"
puts "$rfx::i_freqstart"
puts "***********************************************"
puts "The End Number of Freq Index:"
puts "$rfx::i_freqend"
puts "***********************************************"
puts "The End Number of Bias Index:"
puts "$rfx::i_biasstart"
puts "***********************************************"
puts "The End Number of Bias Index:"
puts "$rfx::i_biasend"

puts "***********************************************"
puts "The 3rd Bias is:"
puts [lindex $rfx::bias 2]

puts "***********************************************"
puts "The 4th Bias is:"
puts [lindex $rfx::freq 3]


rfx::CreateDataset -dataset AC_bias_n@node@ -xaxis "bias" -rfmatrix "AC" 

 
if {[lsearch [list_plots] Plot_CV_bias] == -1} {
	create_plot -1d -name Plot_CV_bias
	link_plots [list_plots] -unlink
	set_plot_prop -title "C<sub>gg</sub>-V<sub>gs</sub> curves (Lg=@Lgate@ <greek>m</greek>m)" \
		-title_font_size 20 -show_legend -show_grid
	set_axis_prop -axis x -title {Gate Voltage [V]} \
		-title_font_size 16 -scale_font_size 14
	set_axis_prop -axis y -title {Capacitance [F]} \
		-scale_format scientific -scale_precision 0 \
		-title_font_size 16 -scale_font_size 14 
}
select_plots Plot_CV_bias


set index  0
set freq [lindex  $rfx::freq $index]
create_curve -dataset AC_bias_n@node@ -axisX "{frequency_${index} bias}" -axisY "{frequency_${index} c(1,1)}" -name curve_Cgg_${index}_n@node@ 
set_curve_prop curve_Cgg_${index}_n@node@  -label "C<sub>gg</sub> Freq=[format %.2e $freq] Hz" \
	  -line_style solid -line_width 3

	  
set index  $rfx::i_freqend 
set freq [lindex  $rfx::freq $index]
create_curve -dataset AC_bias_n@node@ -axisX "{frequency_${index} bias}" -axisY "{frequency_${index} c(1,1)}" -name curve_Cgg_${index}_n@node@ 
set_curve_prop curve_Cgg_${index}_n@node@  -label "C<sub>gg</sub> Freq=[format %.2e $freq] Hz" \
	  -line_style solid -line_width 3


if {[lsearch [list_plots] Plot_CV_bias_all] == -1} {
	create_plot -1d -name Plot_CV_bias_all
	link_plots [list_plots] -unlink
	set_plot_prop -title "C<sub>gg</sub>-V<sub>gs</sub> curves (Lg=@Lgate@ <greek>m</greek>m)" \
		-title_font_size 20 -show_legend -show_grid
	set_axis_prop -axis x -title {Gate Voltage [V]} \
		-title_font_size 16 -scale_font_size 14
	set_axis_prop -axis y -title {Capacitance [F]} \
		-scale_format scientific -scale_precision 0 \
		-title_font_size 16 -scale_font_size 14 
}
select_plots Plot_CV_bias_all	  

for { set index $rfx::i_freqstart} { $index<= $rfx::i_freqend } { incr index } {
	set freq [lindex  $rfx::freq $index]
	create_curve -dataset AC_bias_n@node@ -axisX "{frequency_${index} bias}" -axisY "{frequency_${index} c(1,1)}" -name curve_Cgg_all_${index}_n@node@ 
	set_curve_prop curve_Cgg_all_${index}_n@node@  -label "C<sub>gg</sub> Freq=[format %.2e $freq] Hz" \
		  -line_style solid -line_width 3
}



##### C-Freq at xxx bias (Vg = -2.0 / 2.0 0 )
	  
# create dataset 
rfx::CreateDataset -dataset AC_freq_n@node@ -xaxis "frequency" -rfmatrix "AC" 

if {[lsearch [list_plots] Plot_CV_freq] == -1} {
	create_plot -1d -name Plot_CV_freq
	link_plots [list_plots] -unlink
	set_plot_prop -title "C<sub>gg</sub>-freq curves (Lg=@Lgate@ <greek>m</greek>m)" \
		-title_font_size 20 -show_legend -show_grid
	set_axis_prop -axis x -title {Gate Voltage [V]} \
		-title_font_size 16 -scale_font_size 14
	set_axis_prop -axis y -title {Capacitance [F]} \
		-scale_format scientific -scale_precision 0 \
		-title_font_size 16 -scale_font_size 14 
}
select_plots Plot_CV_freq


set index  0
set bias [lindex  $rfx::bias $index]
create_curve -dataset AC_freq_n@node@ -axisX "{bias_${index} frequency}" -axisY "{bias_${index} c(1,1)}" -name curve_Cgg_Cfreq_${index}_n@node@ 
set_curve_prop curve_Cgg_Cfreq_${index}_n@node@  -label "C<sub>gg</sub> Bias=[format %.2f $bias] V" \
	  -line_style solid -line_width 3


set index  $rfx::i_biasend
set bias [lindex  $rfx::bias $index]
create_curve -dataset AC_freq_n@node@ -axisX "{bias_${index} frequency}" -axisY "{bias_${index} c(1,1)}" -name curve_Cgg_Cfreq_${index}_n@node@ 
set_curve_prop curve_Cgg_Cfreq_${index}_n@node@  -label "C<sub>gg</sub> Bias=[format %.2f $bias] V" \
	  -line_style solid -line_width 3
set_axis_prop -plot Plot_CV_freq -axis x -type log


```

date：20221018
