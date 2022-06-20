#### 1、sde或sdevice中判断语句 tcl command

```tcl
#if [string equal "@break@" "current"]
	BreakCriteria{Current (Contact="drain" absval=1e-8)}
	#else
	BreakAtIonIntegral
        ComputeIonizationIntegrals(WriteAll)
#endif
```
#### 2、多行注释
```tcl
#if 0
注释内容
#endif
```
#### 3、sde或sdevice中判断语句 scheme command
```tcl
(if   (string=?  “@type@”  “NMOS”)|
	(begin
			如果相等执行
			)
	(begin
			如果不相等执行
			)
)
```
#### 4、分区，维持网格一致
```tcl
(sdesnmesh:axisaligned “xCuts” (list  (/  Lgate  -2) (/  Lgate  -2))
(sdesnmesh:axisaligned “yCuts” (list  0 1)
#endif
```
#### 5、三角函数
```tcl
(define TAN (tan (* (/ gate_angle 180) PI) ))
(define COS (cos (* (/ gate_angle 180) PI) ))
(define SIN (sin (* (/ gate_angle 180) PI) ))

#endif
```
#### 6、倒角定义
```tcl
(sdegeo:fillet-2d (list 
	(car (find-vertex-id (position (- (+ Wleft Wgate) (/ Hgate TAN)) Hgate 0)))
	(car (find-vertex-id (position (+ Wleft (/ Hgate TAN)) Hgate 0)))
	) R)

#endif
```
#### 7、返回创建区域的编号，将两个区域合成一个，移动点的位置
```tcl
(define LOCOS1_ID (sdegeo:create-rectangle
		(position 0.0 0.0 0.0 )  (position Wleft Tox_locos 0.0 ) "Oxide"  "R.LOCOS_thin" ) 
)
(define LOCOS2_ID (sdegeo:create-rectangle
	(position 0.0 (- (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 )  (position Wlocos (+ (/ Tox_locos 2) (/ Tlocos 2) ) 0.0 ) "Oxide"  "R.LOCOS_thick" )
)
(define LOCOS_ID (sdegeo:bool-unite (list LOCOS1_ID LOCOS2_ID)) )
(sde:add-material  LOCOS_ID (generic:get LOCOS_ID "material" ) "R.LOCOS")
;(sde:add-material  LOCOS_ID  "Gold" (generic:get LOCOS_ID "region" ))

(sdegeo:move-vertex
		(list 
			(car
				(find-vertex-id (position Wlocos Tox_locos  0.0 ))
			)
			(car
				(find-vertex-id (position Wlocos 0.0        0.0 ))
			)
		)
		(gvector Wneck 0.0 0.0 ) ;direction length
	)

#endif
```
#### 8、定义大致网格范围
```tcl
(define fs 1.0)
(define dx (* fs (/ (+ Wleft Wgate Wright) 10.0)))
(define dy (* fs (/ ymax 10.0)))
(define dTox (* fs (/ Tox_sidewall 3.0) ))
```
#### 9、修改部分区域内材料性质
```tcl
  Region = "xx"
  {  Epsilon
			{
										     #perp. to c-axis
				epsilon	=	@K@			     #[94Nin, 05Pen], 6H-SiC+4H-SiC										     
				#epsilon =	9.66			     #[70Pat], ioffe database,	6H-SiC
				

	}
		}

#endif
```
#### 10、sde和sdevice中的if语句
```tcl
#if [string equal "@type@" "pshield"]

#elif [string equal "@type@" "thick_pshield"]

#else

#endif

#endif
```