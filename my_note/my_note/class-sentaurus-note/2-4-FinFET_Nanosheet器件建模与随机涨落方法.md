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
