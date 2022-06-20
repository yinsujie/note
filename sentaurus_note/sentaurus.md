#### 1、sde或sdevice中判断语句 tdl command

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
