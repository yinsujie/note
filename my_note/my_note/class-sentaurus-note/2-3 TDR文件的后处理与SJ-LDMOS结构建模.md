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
