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
  - [1.6 接口型態](#1.6)
  - [1.7.2 SSD、HDD應用場合](#1.7.2)
- [二、SSD主控和全閃存陣列](#2)
  - [2.1 SSD系統架構](#2.1)


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

電信號兼容性和硬件兼容性

- SSD工作時，主機提供的電信號處於非穩定狀態，比如存在抖動、信號完整性差等情況，但依然在規範誤差範圍內，此時SSD通過自身的硬件設計（比如power regulator）和接口信號完整性設計依然能正常工作，數據也依然能正確收發

容錯處理

- SSD盤若能容錯並返回錯誤狀態給主機，提供足夠的日誌來幫助主機軟硬件開發人員調試就更好了。這裡的錯誤包括接口總線上的數據CRC錯誤、丟包、數據命令格式錯誤、命令參數錯誤等。

<h2 id="1.6">1.6 接口型態</h2>

SSD接口形態和尺寸的英文是SSD Form Factor

![img16](./image/img16.PNG)

mSATA是前些年出現的，與標準SATA相比體積大為縮小，主要應用於消費級筆記本領域。但待M.2出現後，基本上替代了mSATA，革了它的命。

M.2原名是NGFF（Next Generation Form Factor），它是為超極本（Ultrabook）量身定做的新一代接口標準，主要用來取代mSATA接口，具備體積小巧、性能主流等特點。

![img17](./image/img17.PNG)

U.2Form Factor（SFF-8639）起步於PCIe SSD 2.5寸盤形態制定的接口，到後來統一了SATA、SAS和PCIe三種物理接口，從而減小了下游SSD應用場合的接口復雜度，是一種新型連接器FormFactor，目前標準還在更新中。

BGA SSD

- 隨著製程和封裝技術的成熟，當今一個PCB 2.5寸大小的存儲器可以放到一個16mm×20mm BGA封裝中，這就是BGA uSSD

    ![img18](./image/img18.PNG)

- 目前（2017年）世界上最小尺寸的SSD，其與手機中的eMMC及UFS的尺寸一樣大小，但是容量要大得多

    ![img19](./image/img19.PNG)

![img20](./image/img20.PNG)

SDP(SATA Disk in Package)

- 指將SSD主控芯片、閃存芯片在封裝廠封裝成一體化模塊，經過開卡量產、測試後出廠。這種產品形態相當於SSD的半成品，只需要加上外殼就能成為完整的SSD產品

    ![img21](./image/img21.PNG)

與傳統$PCBA$模式相比，$SDP^{TM}$具備哪些優勢？

![img22](./image/img22.PNG)

U.2

- U.2俗稱SFF-8639，這是新生產物，採用非AIC(Add-In Card)形式，以盤的形態存在。開發U.2的目的是統一SAS、SATA、PCIe三種接口，方便用戶部署。

<h2 id="1.7.2">1.7.2 SSD、HDD應用場合</h2>

數據按照熱度的不同會採取不同的存儲方式，這樣可以平衡性能和成本的問題，俗稱性價比。在HDD和SSD二分天下的今天，SSD主要用於存放和用戶貼近的熱數據，其對總容量需求較小，性能優先；HDD主要用於存放和用戶較遠的溫（warm）數據或冷（cold）數據，其對總容量需求較大，價格優先

- 數據加速層：採用PCIe接口的高性能的SSD。
- 熱數據（頻繁訪問）層：採用普通SATA、SAS SSD。
- 溫數據層：採用高性能HDD。
- 冷數據層：採用HDD。
- 歸檔層：採用大容量價格低廉的HDD，甚至磁帶

SSD的研發模式，三方配合：主控廠商 + 閃存廠商 + 生產製造

<h1 id="2">二、SSD主控和全閃存陣列</h1>

SSD主要由兩大模塊構成——主控和閃存介質。
除了上述兩大模塊外，可選的還有緩存單元

主控是SSD的大腦，承擔著指揮、運算和協調的作用

- 實現標準主機接口與主機通信；
- 實現與閃存的通信；
- 運行SSD內部FTL算法。

<h2 id="2.1">2.1 SSD系統架構</h2>

SSD作為數據存儲設備，其實是一種典型的（System on Chip）單機系統，有主控CPU、RAM、操作加速器、總線、數據編碼譯碼等模塊，操作對象為協議、數據命令、介質，操作目的是寫入和讀取用戶數據。

![img23](./image/img23.PNG)

- 前端（Host Interface Controller，主機接口控制器）跟主機打交道，接口可以是SATA、PCIe、SAS等

    ![img24](./image/img24.PNG)

  - SATA的全稱是Serial Advanced Technology Attachment（串行高級技術附件），是一種基於行業標準的串行硬件驅動器接口，是由Intel、IBM、Dell、APT、Maxtor和Seagate公司共同提出的硬盤接口規

    ![img25](./image/img25.PNG)

  - SAS（Serial Attached SCSI）即串行連接SCSI，是新一代的SCSI技術，和現在流行的Serial ATA（SATA）硬盤相同，都是採用串行技術以獲得更高的傳輸速度，並通過縮短連接線改善內部空間等

  - SAS是並行SCSI接口之後開發出的全新接口，此接口的設計是為了改善存儲系統的效能、可用性和擴充性，並且提供與SATA硬盤的兼容性。SAS的接口技術可以向下兼容SATA

    ![img26](./image/img26.PNG)

  - PCIe（Peripheral Component Interconnect Express）是一種高速串行計算機擴展總線標準，旨在替代舊的PCI、PCI-X和AGP總線標準。

  - PCIe屬於高速串行點對點多通道高帶寬傳輸，所連接的設備分配獨享通道帶寬，不共享總線帶寬，主要支持主動電源管理、錯誤報告、端對端的可靠性傳輸、熱插拔以及服務質量（QoS，Quality of Service）等功能。

- 後端（Flash Controller，閃存控制器）跟閃存打交
道並完成數據編解碼和ECC
- 緩衝（Buffer）、DRAM
- 模塊之間通過AXI高速和APB低速總線互聯互通，完成信息和數據的通信
- 固件（Firmware）統一完成SSD產品所需要的功能，調度各個硬件模塊，完成數據從主機端到閃存端的寫入和讀取






