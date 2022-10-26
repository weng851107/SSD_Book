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

