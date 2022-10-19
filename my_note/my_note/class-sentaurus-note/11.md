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
