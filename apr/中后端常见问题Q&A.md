# 中后端常见问题Q&A

---

**Author:** Yujiang Guan 	**Created Date:** 2022-06-29

|Version|Author|Modified|Contents|
| ---------| ---------| ------------| ----------------------------|
|1.0|YJ-Guan|2022-06-29|First Version|
|1.1|YJ-Guan|2023-06-12|Markdown Version, Uploaded|

**1. Encounter导入Innovus导出的DB时出错，报ENCVL-209，.v.bin syntax error**

* **S:** Innovus里导出.v netlist (`saveNetlist`​) 放入DB中并替换.global中的init_verilog设置

**2. Encounter里followpin的via打不上**

* **S:** 多打了一层M3 placement blockage

**3. IO Corner放不到角上**

* **S:** Innovus没有把Corner cell识别成corner，使用 `dbSetIsIoCorner [dbGetIoByName u_CORNERA_[1]] 1`​ 更改Corner属性

**4. Variation中报Followpin到EndCap中出现Open**

* **S:** 检查power连接，via是否都打上，如果确认没问题则该问题可以忽略

**5. CTS长不出来，timingReport里全部为空**

* **S:** 检查mmmc中是否缺少lib

**6. CTS中optDesign修hold时一直iteration**

* **S:** 检查cpf文件中的hold uncertainty和 setOptMode 中的 -holdTargetSlack 是否过大

**7. Calibre中出现大量via错位/Standard cell里报DRC**

* **S:** 检查导出.gds时使用的库.gds文件是否和.lef，layermap为同一种metal option (7m0t1f)

**8. Virtuoso中LVS时，多电压域设计中body连接有问题**

* **S:** 出现该问题原因是Virtuoso可能默认只识别了一种电压的body，可使用SEPGND层，将每个电压域单独框出，并需修改LVS Rule文件，取消#DEFINE SEPARATE_GND的注释，以及更改DEFINE的金属设置。如果存在FILLER对应问题生成Netlist时可以去掉FILLER。出现如VDDIO等对应问题，可以尝试手动在virtuoso里打label

**9. Floorplan时，Core box缩不小，报IMPFP-316: Not enough space for floorplan resize**

* **S:** Source会design_import的DB，再执行一遍floorplan

**10. 提示选择的view不是 active view**

* **S:** 用 set_analysis_view 设置的view 即为active view, report_analysis_views -type active

**11. 04或其他服务器的Tempus和Virtuoso的OA lib有冲突**

* **S:** 在.bashrc中`unset $OA_HOME`​，eda启动时会在自己的根目录下找OA_HOME,但Tempus(SSV)目录下的OA版本不兼容，可以手动在innovus启动的makefile里添加 `export OA_HOME=/workspace/home/guanyj/Software/eda/oa_v22.50.010 &&`​

**12. 出现followpin跳到高层金属穿过macro，连接到穿过macro的stripe上**

* **S:** 关闭sRoute中的Allow Layer Changing，或者在sRoute脚本中加入`-allowLayerChange 0`​

**13. CTS后的时序报告中出现输入CLK Arrival time为负，即Beginpoint Arrival Time为负的情况**

* **S:** CTS后Innovus会自动将in-FF-Latch的路径合为一条reg2reg路径，并给加上负的clk slack，负的slack覆盖第一个in2FF的time。这样可能会同时造成在分析时序时对in2reg路径引入实际不存在的负的clock latency，造成后续流程中EDA过于乐观估计in2reg的hold time 问题且并不去进行short path padding，造成min-delay violation，在/DB/enc.dat/mmmc/view/xxx_view/latency.sdc中删掉negative clock latency 即可

**14. RAM/ROM在LVS时出现IO短路到VSS或VDD**

* **S:** 由于在PR阶段RAM中的FollowPin未包含在.lef文件中，导致在IO端口进行连线的时候可能会造成连线短路到VSS/VDD，需要在RAM上打上ME4 Route Blockage (RAM中GND/VSS对应的金属层) 防止短路，同时注意检查Innovus中出现的Violation

**15. IO Pad的各个使能端如SMT/PIN0/PIN1等没有接到TIEHI/TIELO上，Design Browser 中显示连接到VSS与VDD上，连线时直接接到了Pad的VDD/VSS上**

* **S:** 返回Placement重跑？先定义GlobalNet再定义TIEHI/TIELO？

**16. LVS过程中报Cell内部连接出现问题，如IUMBF中的INVT supply出现问题等**

* **S:** 使用LVS BOX将出问题的标准单元设置为Black Box，具体方法为在Calibre LVS中进入LVS Options-LVS BOX，在上面的框里添加一个Statement，Value写Cell的名字就好，如IUMBFS

**17. 关于Logically equivelant cell**

* **S:** 在APR中定义多个Vt下的时序库时，根据相同的logic function会对.v网表中的cell进行替换，本质上是有相同的FootPrint, reportFootPrint

**18. 在tcl中调用bash shell命令**

* **S:** `exec sh -c {design_vision}`​  #其中design_vision是要执行的shell命令

**19. 出现 Segmentation fault (core dumped)**

* **S:** 检查一下有没有连上外网，或者过一段时间自己就好了，QRC出现这个问题也有可能是stacksize被限制过低，用ulimit -a看一下stacksize，可以在启动innovus前用ulimit -s unlimited 调整stacksize

**20. 需要分散Cell，Cell周围需要留出间隔**

* **S: ​**使用如下指令 `setPlaceMode -place_global_max_density   -uniformDensity true`​， 或添加 particle place blockage（加Placement Blockage，然后更改为Paartial）

       TSMC 28的设计中，DFF的CK端口附近可能出现DRC问题，可对DFF均添加Padding，如：

       `specifyCellPad DF* -left 200 -top 200 -bottom 200 -right 200`​

  或 `specifyCellPad DFD* 2`​

**21. 如何改变Instance Status？**

* **S:** 使用`setInstancePlacementStatus / placeInst`​ 指令

**22. 有时候调用完mmmc.tcl后，innovus里mmmc配置没有改变**

* **S:** 进GUI界面Timing-MMMC Browser中点Reset刷新一下就好了

**23. 出现IQRC时，分配多线程任务失败，出现 couldn't fork child proces- ​******S:******​ resource temporarily unavailable 导致Innovus崩溃**

* **S:** 例如HSPICE/GENUS在跑完之后，如果没关闭Terminal，它们仍会在后台占用线程，关闭启动它们的Terminal后重试

**24. 如何从Virtuoso中导出.gds**

* **S:** UMC55的设计：File-Export-Stream Technology Library选择UMC55LP，下方Layer选项导入/technology/umc/55nm_201908/pdk/FDK/stream.map

**25. Innovus将Error Detection FF识别成了Latch**

* **S:** 在K出来的.lib文件中，给pin(Q)加function : "IQ"; pin(QN) 加function : "IQN";  
  cell里加

```verilog
    ff (IQ,IQN) {
      clocked_on : "CLK";
      next_state : "D";
      power_down_function : "(!VDD) + (VSS)";
    }
```

具体可参照标准单元库里的FF

**26. Stripe接不到SRAM的Block Ring上**

* **S:** 用Encounter试试 （仅限UMC55的设计，TSMC28的可以用Innovus直接连）

**27. 需要将Macro的VDD/VSS连接到Macro的Block Ring上**

* **S:** 需要在sRoute中使用BlockPin，且Advanced中Block Pin-Pin Selection-勾上All Pins，Target Selection勾上Block Ring，具体可以参考10. 注意T28的SRAM自身不带Power Ring，需要给他打电源

**28. 如何生成qrcmapfile/qrclayermap？**

* **S:**

```tcl
setExtractRCMode -engine postRoute -coupled true -effortLevel high 
extractRC
```

这时Innovus会在当前工作目录下生成一个extLogDir，在里面可以找到自动生成的layer map 信息，extLogDir/IQRC_*/.layermap.log  
注意TSMC28的设计在Route中跑QRC的时候不需要设置setExtractRCMode -lefTechFileMap ./qrc/qrclayermap  
同时注意，在把TSMC28的设计的GDSII导入Virtuoso时，也不需要Load Mapfile，否则DRC会报奇怪的错误

**29. 28nm工艺中不允许使用Filler1？**

* **S:** 由于OD-Spacing Effect不能使用Filler1，在Place阶段设置约束使两个Cell至少保持2个Site的Gap  
  ​`setPlaceMode -fillerGapEffort high -fillerGapMinGap 0.28`​

**30. Route完出现很多DRC Violation？**

* **S:** a. 太多了的话说明Congestion严重，用以下指令摆开一些

  ```tcl
  setPlaceMode -congEffort high
  setPlaceMode -uniformDensity true
  setPlaceMode -place_global_max_density 0.70
  setPlaceMode -modulePadding
  ```

  b. 用editDeleteViolations删掉有DRC的net，再ecoRoute  
  c. 用specifyCellPad DFD* 2给出问题的Cell加Pad，隔开一些空间方便走线，再ecoPlace、ecoRoute  
  d. 有时也可以直接手动挪开一些Cell，留出空隙给走线，注意对齐Size，最后再ecoRoute  
  e. 重做Floorplan

**31. Setup难修，或者setup hold互相卡？**

* **S:** 对于一块区域的Setup，Hold难修，多半是这块Density太大了，参考#30的解决办法，或者  
  ​`setPlaceMode -place_global_module_padding inst_SSCNN/inst_GB_Top/inst_GBB 1.4`​，给整个Module设置Padding  
  对于Setup Hold相互卡的情况，如果多次OPT还不能解决，可以尝试在一次optDesign中一起加入-setup -hold ，有效果。

**32. 对不同电压域插入WELLTAP或ENDCAP时，报错ENCSP-5170/IMPSP-5170，提示目标电压域里没有Module，无法插入WELLTAP/ENDCAP**

* **S:** 在Elabration之后，和综合完成以后，输出cpf以前使用  
  ​`set_attr ui_respects_preserve falseungroup -all`​  
  将层次Flatten即可，两个位置都需要加上，不然没用

‍
