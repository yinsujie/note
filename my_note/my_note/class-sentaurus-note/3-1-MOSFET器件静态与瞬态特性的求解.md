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
