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
