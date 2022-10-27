```
Author: Antony_Weng <weng851107@gmail.com>

This file is only used for the record of the learning process, only used by myself and the file has never been leaked out.
If there is related infringement or violation of related regulations, please contact me and the related files will be deleted immediately. Thank you!
```

# 深入浅出ssd-固态存储核心技术原理与实战

- [深入浅出ssd-固态存储核心技术原理与实战.pdf](./深入浅出ssd-固态存储核心技术原理与实战.pdf)

    ![深入浅出ssd.jpg](./深入浅出ssd.jpg)

- [一、SSD總述](#1)
  - [1.1 引言](#1.1)
  - [1.2 SSD vs. HDD](#1.2)
  - [1.4 SSD 基本工作原理](#1.4)
  - [1.5.1 基本信息剖析](#1.5.1)
  - [1.5.2 性能剖析](#1.5.2)
  - [1.5.3 壽命（Endurance）剖析](#1.5.3)
  - [1.5.4 數據可靠性剖析](#1.5.4)
  - [1.5.5 功耗剖析](#1.5.5)


<h1 id="1">一、SSD總述</h1>

SSD（Solid State Drive），即固態硬盤，是一種以半導體閃存（NAND Flash）作為介質的存儲設備

<h2 id="1.1">1.1 引言</h2>

SATA2.5吋與3.5吋，以及M.2接口的尺寸
Note: 傳統HDD2.5吋

![img00](./image/img00.PNG)

SSD是用固態電子存儲芯片陣列製成的硬盤，主要部件為控制器和存儲芯片

SSD硬件包括幾大組成部分：主控、閃存、緩存芯片DRAM（可選，有些SSD上可能只有SRAM，並沒有配置DRAM）、PCB（電源芯片、電阻、電容等）、接口（SATA、SAS、PCIe等），其主體就是一塊PCB

![img01](./image/img01.PNG)

SSD內部運行固件（Firmware，FW）負責調度數據從接
口端到介質端的讀寫，還包括嵌入核心的閃存介質壽命和可靠性管理調度算法，以及其他一些SSD內部算法

SSD的三大技術核心 = SSD控制器 + 閃存 + 固件

### 儲存介質

存儲介質按物理材料的不同可分為三大類：光學存儲介質、半導體存儲介質和磁性存儲介質

- 光學存儲介質，就是大家之前都使用過的DVD、CD等光盤介質，靠光驅等主機讀取或寫入
- HDD是以磁性存儲介質來存儲數據
- SSD採取半導體芯片作為存儲介質

![img02](./image/img02.PNG)

閃存

- 浮柵晶體管（Floating Gate Transistor）
- 相比MOSFET就多了個Floating Gate，懸浮在中間，所以叫浮柵
- 它被高阻抗的材料包裹，和上下絕緣，能夠保存電荷，而電荷通過量子隧道效應進入浮柵

    ![img08](./image/img08.PNG)

<h2 id="1.2">1.2 SSD vs. HDD</h2>

HDD = 馬達 + 磁頭 + 磁盤, 機械結構
SSD = 閃存介質 + 主控, 半導體存儲芯片結構

![img03](./image/img03.PNG)

![img04](./image/img04.PNG)

![img07](./image/img07.PNG)

1. 性能好

- 性能測試工具包括 連續讀寫吞吐量（Throughput）工具 和 隨機讀寫IOPS工具 兩種

    ![img05](./image/img05.PNG)

2. 功耗低

- 工作功耗HDD為6～8W，SATA SSD為5W，待機功耗SSD可降低到毫瓦（mW）級別

    ![img06](./image/img06.PNG)

- 科學地比較功耗的方法應該是Power/IOPS，也就是比較單位IOPS性能上的功耗輸出，該值越低越好

3. 抗震防摔

- SSD內部不存在任何機械部件，相比HDD更加抗震

4. 無噪聲

- 客觀上，由於結構上沒有馬達的高速運轉，SSD是靜音的

5. 尺寸小巧多樣

- HDD一般只有3.5寸和2.5寸兩種形式

- SSD除了3.5寸和2.5寸，還有更小的可以貼放在主板上的M.2形式，甚至可以小到芯片級，例如BGASSD的大小只有16mm×30mm

<h2 id="1.4">1.4 SSD 基本工作原理</h2>

從主機PC端開始，用戶從操作系統應用層面對SSD發出請求，文件系統將讀寫請求經驅動轉化為相應的符合協議的讀寫和其他命令，SSD收到命令執行相應操作，然後輸出結果

SSD的輸入是 命令（Command），輸出是 數據（Data） 和 命令狀態（Command Status）

![img09](./image/img09.PNG)

SSD主要有三大功能模塊組成：

- 前端接口和相關的協議模塊
- 中間的FTL層（Flash Translation Layer）模塊
- 後端和閃存通信模塊

SSD通過諸如SATA、SAS和PCIe等接口與主機相連，實現對應的ATA、SCSI和NVMe等協議

![img10](./image/img10.PNG)

SSD是怎麼進行讀寫的，以寫為例

- 主機通過接口發送寫命令給SSD，SSD接收到該命令後執行，並接收主機要寫入的數據
- 數據一般會先緩存在SSD內部的RAM中，FTL會為每個邏輯數據塊分配一個閃存地址，當數據湊到一定數量後，FTL便會發送寫閃存請求給後端，然後後端根據寫請求，把緩存中的數據寫到對應的閃存空間。
- SSD內部維護了一張邏輯地址到物理地址轉換的映射表

由於閃存不能覆蓋寫，閃存塊需擦除才能寫入

主機發來的某個數據塊，它不是寫在閃存固定位置，SSD可以為其分配任何可能的閃存空間寫入。透過 FTL 完成邏輯數據塊到閃存物理空間的轉換或者映射。

- 假設SSD容量為128GB，邏輯數據塊大小為4KB，所
以該SSD一共有128GB/4KB=32M個邏輯數據塊
- 每個邏輯塊都有一個映射，即每個邏輯塊在閃存空間都有一個存儲位置
- 閃存地址大小如果用4字節表示，那麼存儲32M個邏輯數據塊在閃存中的地址則需要32M×4B=128MB大小的映射表。

一個SSD在前端協議及閃存確定下來後，差異化就體現在FTL算法上了。 FTL算法決定了性能、可靠性、功耗等SSD的核心參數。

<h2 id="1.5.1">1.5.1 基本信息剖析</h2>

### SSD容量

以字節（Byte）為單位

同樣一組數據，二進制比十進制會多出7%的容量，例如：

- 十進制128GB：128×1000×1000×1000=128000000000字節
- 二進制128GB：128×1024×1024×1024=137438953472字節

裸容量：以二進制為單位的容量
用戶容量：以十進制為單位的容量

SSD可以利用這多出來的7%空間管理和存儲內部數據，比如把這部分額外的空間用作FTL映射表存儲空間、垃圾回收所需的預留交換空間、閃存壞塊的替代空間等。

7%多餘空間也可以轉換為OP概念（Over Provisioning）

$$
OP=\frac{(SSD裸容量-用戶容量)}{用戶容量}
$$

### 介質信息

閃存分SLC、MLC、TLC（甚至QLC），它指的是一個存儲單元存儲的比特數
- SLC（Single-Level Cell）即單個存儲單元存儲1bit的數據。 SLC速度快，壽命長（5萬～10萬次擦寫壽命），但價格超貴（約是
MLC 3倍以上的價格）。
- MLC（Multi-Level Cell）即單個存儲單元存儲2bit的數據。 MLC速度一般，壽命一般（約為3k～10k次擦寫壽命），價格一般。
- TLC（Trinary-Level Cell）即單個存儲單元存儲3bit的數據，也
有閃存廠家叫8LC，速度慢，壽命短（約500～1500次擦寫壽命），
價格便宜。

    ![img11](./image/img11.PNG)

### 外觀尺寸

![img12](./image/img12.PNG)

<h2 id="1.5.2">1.5.2 性能剖析</h2>

### 性能指標

IOPS（Input Output Operations Per Second，反映的是隨機讀寫性能）
- 設備每秒完成IO請求數，一般是小塊數據讀寫命令的響應次數，比如4KB數據塊尺寸。 IOPS數字越大越好。

吞吐量（Throughput，單位 MB/s，反映的是順序讀寫性能）：
- 每秒讀寫命令完成的數據傳輸量，也叫帶寬（Bandwidth），一般是大塊數據讀寫命令，比如512KB數據塊尺寸。吞吐量越大越好。

Response Time/Latency（響應時間/時延，單位ms或μs）：

- 每個命令從發出到收到狀態回复所需要的響應時間，時延指標有平均時延（Average Latency）和最大時延兩項（Max Latency）。響應時間越小越好。

### 訪問模式

Random/Sequential：隨機（Random）和連續（Sequential）數據命令請求。
- 何為隨機和連續？指的是前後兩條命令LBA地址是不是連續的

Block Size：塊大小，即單條命令傳輸的數據大小，性能測試從4KB～512KB不等。
- 隨機測試一般用小數據塊，比如4KB；
- 順序測試一般用大塊數據，比如512KB。

Read/Write Ratio：讀寫命令數混合的比例

<h2 id="1.5.3">1.5.3 壽命（Endurance）剖析</h2>

衡量SSD壽命主要有兩個指標，

- DWPD（Drive Writes Per Day）：即在SSD保質期內，用戶每天可以把盤寫滿多少次；
- TBW（Terabytes Written）：在SSD的生命週期內可以寫入的總的字節數。

200GB SSD五年使用期限內對應的壽命是3600TB，平均到每天可以寫入3600TB/（5×365）=1972GB，這塊盤本身200GB，1972GB相當於每天寫入10次，也就是規范書說的10Drive Writes Per Day，簡稱10DWPD。 DWPD為5年的壽命期內每天可以滿盤寫入的次數。

TBW就是在SSD的生命週期內可以寫入的總的字節數

$$
總寫入量TBW=Capacity(單盤容量)\frac{NAND PE Cycles(NAND寫擦除壽命)}{WA(寫放大)}
$$

- NAND PE Cycles：SSD使用的閃存標稱寫擦除次數，如3K、5K。
- Capacity：SSD單盤用戶可使用容量。
- WA：寫入放大係數，這跟SSD FW的設計和用戶的寫入的數據類型（順序寫還是隨機寫）強相關。

TBW和DWPD的計算公式：

$$
DWPD=\frac{TBW}{Years(SSD盤標示使用年限)*365*Capacity(單盤容量)}
$$

<h2 id="1.5.4">1.5.4 數據可靠性剖析</h2>

衡量SSD可靠性的關鍵指標

- UBER：Uncorrectable Bit Error Rate，不可修復的錯誤比特率
  - UBER是一種數據損壞率衡量標準，等於在應用了任意特定的錯誤糾正機制後依然產生的每比特讀取的數據錯誤數量佔總讀取數量的比例（概率）
- RBER：Raw Bit Error Rate，原始錯誤比特率
  - RBER反映的是閃存的質量。所有閃存出廠時都有一個RBER指標。 RBER指標也不是固定不變的，閃存的數據錯誤率會隨著使用壽命（PE Cycle）的增加而增加

    ![img13](./image/img13.PNG)

  - RBER還跟閃存內部結構也有關係，Upper Page的RBER比Lower Page的RBER要高兩個數量級。

    ![img14](./image/img14.PNG)

- MTBF：Mean Time Between Failure，平均故障間隔時間
  - MTBF指標反映的是產品的無故障連續運行時間


閃存有天然的數據比特翻轉率

- 擦寫磨損（P/E Cycle）
- 讀取乾擾（Read Disturb）
- 編程干擾（Program Disturb）
- 數據保持（Data Retention）發生錯誤

<h2 id="1.5.5">1.5.5 功耗剖析</h2>

SSD的功耗類型：

- 空閒（Idle）功耗：
  - 當主機無任何命令發給SSD，SSD處於空閒狀態但也沒有進入省電模式時，設備所消耗的功耗。
- Max active功耗：
  - 最大功耗是SSD處於最大工作負載下所消耗的功耗，SSD的最大工作負載條件一般是連續寫
- Standby/Sleep功耗：
  - 在Standby和Sleep狀態下，設備應盡可能把不工作的硬件模塊關閉，降低功耗。一般消費級SSD Standby和Sleep功耗為100～500mW。
- DevSleep功耗：
  - SATA和PCIe新定義的一種功耗標準，目的是在Standby和Sleep基礎上再降一級功耗
  - 配合主機和操作系統完成系統在休眠狀態下（如Hibernate），SSD關掉一切自身模塊，處於極致低功耗模式，甚至是零功耗。一般是10mW以下。

對於主機而言，它的功耗狀態和SSD作為設備端是一一對應的，而功耗模式發起端是主機，SSD被動執行和切換對應功耗狀態。

系統Power State（SATA SSD作為OS盤）：

- S0：工作模式
- S1：是低喚醒延遲的狀態, 硬件負責維持所有的系統上下文。
- S2：與S1相似，從處理器的reset vector開始執行。
- S3：睡眠模式（Sleep），CPU不運行指令，SATA SSD關閉，
除了內存之外的所有上下文都會丟失。
- S4：休眠模式（Hibernation），CPU不運行指令，SATA SSD
關閉，DDR內容寫入SSD中，所有的系統上下文都會丟失，OS負責上
下文的保存與恢復。
- S5：Soft off state，與S4相似，但OS不會保存和恢復系統上下
文。消耗很少的電能，可通過鼠標鍵盤等設備喚醒。

SSD功耗最大的是ASIC主控和閃存模塊

- 特定應用積體電路（英語：Application Specific Integrated Circuit，縮寫：ASIC）
- 控制SSD溫度是固件設計要考慮的，就是設計降溫處理算法

    ![img15](./image/img15.PNG)

<h2 id="1.5.6">1.5.6 SSD系統兼容性</h2>

BIOS和操作系統的兼容性

- SSD上電加載後，主機中的BIOS作為第一層軟件和SSD進行交互。第一步，和SSD發生鏈接，SATA和PCIe走不同的底層鏈路鏈接

- 主機端和SSD連接成功後發出識別盤的命令（如SATA Identify）來讀取盤的基本信息，基本信息包括產品part number、FW版本號、產品版本號等，BIOS會驗證信息的格式和數據的正確性，

- 接著BIOS會讀取盤其他信息，如SMART，直到BIOS找到硬盤上的主引導記錄MBR，加載MBR
  - S.M.A.R.T指的是自我監控分析與報告技術，也是一種內建於硬碟和 SSD 中的監控功能
  - 主開機紀錄（Master Boot Record，縮寫：MBR），又叫做主引導磁區，是電腦開機後存取硬碟時所必須要讀取的首個磁區

- MBR開始讀取硬盤分區表DPT，找到活動分區中的分區引導記錄PBR(Partition Boot Record)，並且把控制權交給PBR……最後，SSD通過數據讀寫功能來完成最後的OS加載

啟動順序： BIOS -> MBR -> DPT -> PBR

從測試角度來看，系統兼容性認證包括以下各個方面：

- OS種類（Windows、Linux）和各種版本的OS；
- 主板上CPU南北橋芯片組型號（Intel、AMD）和各個版本；
- BIOS的各個版本；
- 特殊應用程序類型和各個版本（性能BenchMark工具、Oracle數據庫……）



