
Ls 系列最適合用於需要低延遲本機儲存體的工作負載，例如 NoSQL 資料庫 (如 Cassandra、MongoDB、Cloudera 和 Redis)。 Ls 系列使用[Intel® Xeon® 處理器 E5 v3 系列](http://www.intel.com/content/www/us/en/processors/xeon/xeon-e5-solutions.html)，提供最多 32 個 CPU 核心。 這是與 G/GS 系列相同的 CPU 效能，而且每個 CPU 核心會有 8 GiB 的記憶體。  

## <a name="ls-series"></a>Ls 系列

ACU：180 - 240
 
| 大小          | CPU 核心 | 記憶體：GiB | 本機 SSD: GiB | 最大資料磁碟 | 最大快取的磁碟輸送量︰IOPS / MBps (快取大小以 GiB 為單位) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC / 網路頻寬 | 
|---------------|-----------|-------------|--------------------------|----------------|-------------------------------------------------------------|-------------------------------------------|------------------------------| 
| Standard_L4s  | 4    | 32   | 678   | 8              | NA / NA (0)          | 5,000 / 125                               | 2 / 高       | 
| Standard_L8s  | 8    | 64   | 1,388 | 16             | NA / NA (0)          | 10,000 / 250                              | 4 / 非常高  | 
| Standard_L16s | 16   | 128  | 2,807 | 32             | NA / NA (0)          | 20,000 / 500                              | 8 / 極高 | 
| Standard_L32s** | 32 | 256  | 5,630 | 64             | NA / NA (0)          | 40,000 / 1,000                            | 8 / 極高 | 
 
MBps = 每秒 10^6 位元組，而 GiB = 1024^3 位元組。 

*Ls 系列 VM 的最大磁碟輸送量 (IOPS 或 MBps)，可能會受到所連結磁碟的數量、大小和串接所限制。 如需詳細資訊，請參閱 [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../articles/storage/storage-premium-storage.md)。 

**執行個體會隔離至單一客戶專用的硬體。