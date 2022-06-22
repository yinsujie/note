## 1、sde或sdevice中判断语句 tcl command

```tcl
#if [string equal "@break@" "current"]
	BreakCriteria{Current (Contact="drain" absval=1e-8)}
	#else
	BreakAtIonIntegral
        ComputeIonizationIntegrals(WriteAll)
#endif
```
## 2、多行注释
```tcl
#if 0
注释内容
#endif
```
## 3、sde或sdevice中判断语句 scheme command
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
## 4、分区，维持网格一致
```tcl
(sdesnmesh:axisaligned “xCuts” (list  (/  Lgate  -2) (/  Lgate  -2))
(sdesnmesh:axisaligned “yCuts” (list  0 1)
#endif
```
## 5、三角函数
```tcl
(define TAN (tan (* (/ gate_angle 180) PI) ))
(define COS (cos (* (/ gate_angle 180) PI) ))
(define SIN (sin (* (/ gate_angle 180) PI) ))
```
## 6、倒角定义
```tcl
(sdegeo:fillet-2d (list 
	(car (find-vertex-id (position (- (+ Wleft Wgate) (/ Hgate TAN)) Hgate 0)))
	(car (find-vertex-id (position (+ Wleft (/ Hgate TAN)) Hgate 0)))
	) R)

```
## 7、返回创建区域的编号，将两个区域合成一个，移动点的位置
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

```
## 8、定义大致网格范围
```tcl
(define fs 1.0)
(define dx (* fs (/ (+ Wleft Wgate Wright) 10.0)))
(define dy (* fs (/ ymax 10.0)))
(define dTox (* fs (/ Tox_sidewall 3.0) ))
```
## 9、修改部分区域内材料性质
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
## 10、sde和sdevice中的if语句
```tcl
#if [string equal "@type@" "pshield"]

#elif [string equal "@type@" "thick_pshield"]

#else

#endif

#endif
```
## 11、Svisual
```tcl
#setdep  @previous@
建立依赖，前面不运行，svisual就不运行

# dataset 
load_file ./IdVg_n@node|IdVg@_des.plt -name  dataset_IdVg_n@node@
加载数据

# plot
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

select_plots {plot_IdVg}

# create absIds variable
create_variable -name absIds -dataset dataset_IdVg_n@node@ -function {abs(<drain TotalCurrent:dataset_IdVg_n@node@>)}
# create a curve 
create_curve -plot plot_IdVg -dataset {dataset_IdVg_n@node@} -axisX {gate OuterVoltage} -axisY {absIds} -name curve_IdVg_n@node@

# compute Gm
create_variable -name Gm -dataset dataset_IdVg_n@node@ -function {abs(diff(<drain TotalCurrent:dataset_IdVg_n@node@>,<gate OuterVoltage:dataset_IdVg_n@node@>))}
create_curve -plot plot_IdVg -dataset {dataset_IdVg_n@node@} -axisX {gate OuterVoltage} -axisY2 {Gm} -name curve_Gm_n@node@


set_curve_prop -plot plot_IdVg {curve_IdVg_n@node@} -label "I<sub>DS</sub>V<sub>GS</sub>:$legend_text_curve"
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -label "G<sub>m</sub>:$legend_text_curve"
set_axis_prop -plot plot_IdVg -axis y -type log



# line width 
set_curve_prop -plot plot_IdVg {curve_IdVg_n@node@} -line_width 3
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -line_width 3 
set_curve_prop -plot plot_IdVg {curve_Gm_n@node@} -line_style dash


# export xy data to csv files
export_curves {curve_IdVg_n@node@ curve_Gm_n@node@ } -plot plot_IdVg -filename ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.csv -format csv  -overwrite
存储csv文件

# export curve plot to png file
export_view ./exported_data/IdVg/n@node@_@type@_Lgate@Lgate@_Tox@Tox@.png -plots {plot_IdVg} -format png -overwrite
存贮png文件

```
## 12、区域之间的界面电荷
```tcl
RegionInterface = "Region1/Region0" { Insert = "Oxide%Silicon.par" }
```
## 13、关于击穿电压收敛性问题
```tcl
Math {
	ElementVolumeAvalanche   用截断体积，计算雪崩产生的贡献
	AvalFlatElementExclusion= 1.5   忽略掉一些过于扁平的单元
	AvalDensGradQF   使用于功率器件中网格密集和收敛性变差的情况
}
Avalanche(Hatakeyama)考虑热模型对器件击穿的影响。

#精度由80变为128，时间变长，容易收敛。
#	ErrEff(electron)= 1e4
#	ErrEff(hole)= 1e4
参考误差，误差约小，收敛性越好，时间越长

```
## 14、offset网格设置
```tcl
(define nlevels 10)	#设置几层曲线
(define factor 1.5)	#曲面增长速率

(sdedr:offset-block "material" "SiliconCarbide"  "maxlevel" nlevels)
(sdedr:offset-block "material" "Oxide"  "maxlevel" nlevels)
;(sdedr:offset-block "region" "R.Si"  "maxlevel" nlevels)

(sdedr:offset-interface "material" "SiliconCarbide" "Oxide" "hlocal" 0.001 "factor" factor)
(sdedr:offset-interface "material" "Oxide" "SiliconCarbide"  "hlocal" 0.001 "factor" factor)

```
## 15、电极定义中Resist的值会除AF

## 16、读取电容值的svisual代码：
```tcl
###Cgs
create_curve -name Cgs($N) -plot Plot_C -dataset PLT_C($N) \
	-axisX v(D) -axisY c(G,S)

set Cgs [probe_curve Cgs($N) -plot Plot_C -valueX 800]
puts "DOE: Cgs [format %.5e [expr abs($Cgs)]]"
remove_curves -plot Plot_C Cgs($N)

###Cgd
create_curve -name Cgd($N) -plot Plot_C -dataset PLT_C($N) \
	 -axisX v(D) -axisY c(G,D)

set Cgd [probe_curve Cgd($N) -plot Plot_C -valueX 800]
puts "DOE: Cgd [format %.5e [expr abs($Cgd)]]"
remove_curves -plot Plot_C Cgd($N)

###Cds
create_curve -name Cds($N) -plot Plot_C -dataset PLT_C($N) \
	 -axisX v(D) -axisY c(D,S)

set Cds [probe_curve Cds($N) -plot Plot_C -valueX 800]
puts "DOE: Cds [format %.5e [expr abs($Cds)]]"
remove_curves -plot Plot_C Cds($N)

###Ciss
create_curve -name Ciss($N) -plot Plot_C -dataset PLT_C($N) \
	-axisX v(D) -axisY c(G,G)

set Ciss [probe_curve Ciss($N) -plot Plot_C -valueX 800]
puts "DOE: Ciss [format %.5e [expr abs($Ciss)]]"
remove_curves -plot Plot_C Ciss($N)

###Coss
create_curve -name Coss($N) -plot Plot_C -dataset PLT_C($N) \
	-axisX v(D) -axisY c(D,D)

set Coss [probe_curve Coss($N) -plot Plot_C -valueX 800]
puts "DOE: Coss [format %.5e [expr abs($Coss)]]"
remove_curves -plot Plot_C Coss($N)

## To Export into an EPS file
## export_view Fig_n@node@_IdVd.eps -plots Plot_IdVd -format eps –overwrite


```
## 17、sdevice中电源的使用方法
```tcl
# power supply                  V+ V-   #instance parameter 
Vsource_pset Vdd 	(dd 0) 	{   dc = 0.0    } #instance parameter 
Vsource_pset Vss 	(ss 0) 	{   dc = 0.0      } #instance parameter 

#     Vinital（初始值）  Vpulse （目标值）      delay （延时时间）  rt（上升时间）            rf（下降时间）         pulse width（脉宽长度）     period （周期时间） 一直循环至仿真最终时间
Vsource_pset Vin 	(in 0) 	{ pulse= (   0     2.0  @<2*time_ps>@  @<0.2*time_ps>@  @<0.2*time_ps>@   @<12*time_ps>@   @<24.4*time_ps>@)
#equal to : [expr 0.2*$time_ps]    }

```
## 18、Math中设置导出错误
```tcl
CNormPrint
NewtonPlot(
Error MinError
Residual
)

```
## 19、导出多张图 
```tcl
# step 1 get the file list 
#得到一个升序排列的文件列表
set FileList [lsort -ascii -increasing [glob -type f profile_n@node|UIS@_*_IGBT_des.tdr]]
puts "Find the TDR files:-----------"
foreach file $FileList { puts "$file" }

# step 2 start movie
set i 0
 
foreach file $FileList {

#load file
load_file "$file" -name dataset_tdr_$i
create_plot -dataset dataset_tdr_$i -name plot_tdr_$i
select_plots "{plot_tdr_$i}" 
 
set_plot_prop -plot "{plot_tdr_$i}" -not_axes_interchanged
set_material_prop {DepletionRegion} -plot "{plot_tdr_$i}" -geom dataset_tdr_$i -off
set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -show_bands
set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -min 0.1 -min_fixed
set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -max 20000 -max_fixed
 
set_legend_prop -plot "{plot_tdr_$i}" -position {0.8 0.750604} -size {0.230597 0.550604}
set_legend_prop -plot "{plot_tdr_$i}" -position {0.75 0.751}
set_legend_prop -plot "{plot_tdr_$i}" -position {0 0.751}
set_legend_prop -plot "{plot_tdr_$i}" -position {0 0}
zoom_plot -plot "{plot_tdr_$i}" -reset	

export_view ./exported_data/UIS/conc@conc@/current_density-conc@conc@/CurrentDensity-conc@conc@-$i.png -plots "{plot_tdr_$i}" -format png -resolution 2906x828

#unload file
remove_plots "{plot_tdr_$i}" 
remove_datasets dataset_tdr_$i
# next i
puts "done! $i"
incr i
}
\

```
## 20、LINUX系统设置快捷方式
```tcl
Pwd；获取文件路径
Ln -s 原路径 新路径（桌面路径） 
```
## 21、输出某一电压下的图片
```tcl
Plot(FilePrefix= “xxxxx”  when ( contact= “drain” Voltage=10 ) NoOverwrite)
输出在drain电极为10V时的电压图片。

```
## 22、利用sort排序
```tcl
对all进行操作
```
## 23、提取QG（读取栅电荷）
```tcl
set Q1 [probe_curve curve_drain_n@node@ -plot plot_QG -valueY 776] 
set Q2 [probe_curve curve_drain_n@node@ -plot plot_QG -valueY 2] 
puts "DOE: Qgd [format %.1f [expr ($Q2-$Q1) ]]" 

```
## 24、提取某一材料/区域中的最大场强
```tcl
load_file ./n@node|BV1200V@_des.tdr -name plot_BV1200V_tdr
create_plot -dataset plot_BV1200V_tdr -name plot_BV1200V
select_plots {plot_BV1200V}
set max [lindex [calculate_field_value -plot plot_BV1200V -geom plot_BV1200V_tdr -field ElectricField -max -ranges {0 3.1 0 11.7 0 0} -materials {Oxide}] 0]

puts "DOE: MAX [format %.2e $max]" 
```


## 25、批量提取图片
```tcl
set i 0

foreach file $FileList {
	
	#load file
	load_file "$file" -name dataset_tdr_$i
	create_plot -dataset dataset_tdr_$i -name plot_tdr_$i
	select_plots "{plot_tdr_$i}" 

	set_plot_prop -plot "{plot_tdr_$i}" -not_axes_interchanged
	set_material_prop {DepletionRegion} -plot "{plot_tdr_$i}" -geom dataset_tdr_$i -off
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -show_bands
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -min 0.1 -min_fixed
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i TotalCurrentDensity -max 20000 -max_fixed

	set_legend_prop -plot "{plot_tdr_$i}" -position {0.8 0.750604} -size {0.230597 0.550604}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0.75 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0}
	zoom_plot -plot "{plot_tdr_$i}" -reset	
	
	export_view ./exported_data/UIS/conc@conc@/current_density-conc@conc@/CurrentDensity-conc@conc@-$i.png -plots "{plot_tdr_$i}" -format png -resolution 2906x828
	
	
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  ElectricField -show_bands
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  ElectricField -min 0 -min_fixed
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  ElectricField -max 200000 -max_fixed

	set_legend_prop -plot "{plot_tdr_$i}" -position {0.8 0.750604} -size {0.230597 0.550604}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0.75 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0}
	zoom_plot -plot "{plot_tdr_$i}" -reset	
	
	export_view ./exported_data/UIS/conc@conc@/ElectricField-conc@conc@/ElectricField-conc@conc@-$i.png -plots "{plot_tdr_$i}" -format png -resolution 2906x828
	
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  LatticeTemperature -show_bands
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  LatticeTemperature -min 300 -min_fixed
	set_field_prop -plot "{plot_tdr_$i}" -geom dataset_tdr_$i  LatticeTemperature -max 360 -max_fixed

	set_legend_prop -plot "{plot_tdr_$i}" -position {0.8 0.750604} -size {0.230597 0.550604}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0.75 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0.751}
	set_legend_prop -plot "{plot_tdr_$i}" -position {0 0}
	zoom_plot -plot "{plot_tdr_$i}" -reset	
	
	export_view ./exported_data/UIS/conc@conc@/LatticeTemperature-conc@conc@/LatticeTemperature-conc@conc@-$i.png -plots "{plot_tdr_$i}" -format png -resolution 2906x828
	
	
	#unload file
	remove_plots "{plot_tdr_$i}" 
	remove_datasets dataset_tdr_$i
	# next i
	puts "done! $i"
	incr i
	
}
```