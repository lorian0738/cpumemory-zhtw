# 3. CPU 快取

比起僅僅 25 年前的 CPU，現今的 CPU 複雜得多了。在那時，CPU 核心的頻率與記憶體匯流排在相同的等級。記憶體存取只比暫存器存取慢了一點點。但這點在 90 年代初期大大地改變了，那時 CPU 設計者提升了 CPU 核心的頻率，但記憶體匯流排的頻率以及 RAM 晶片的效能並沒有等比例地成長。這不是因為無法發展出更快的 RAM，而是如前一節所解釋的。這是可能的，但並不經濟。與當前 CPU 核心一樣快的 RAM，比起任何動態 RAM 都要貴上好幾個數量級。

一台有著非常小、非常快的 RAM 的機器，以及一台有著許多相對快速的 RAM 的機器，如果要在兩者間擇一，在給定超過小小的 RAM 容量的工作集（working set）大小、以及存取硬碟這類次級儲存（secondary storage）媒體的成本之後，後者永遠是贏家。這裡的問題在於次級儲存裝置––通常是硬碟––的速度，它必須用以保存部分被移出（swap out）的工作集。存取這些硬碟甚至比 DRAM 存取要慢上好幾個數量級。

幸運的是，不必做出非全有即全無的（all-or-nothing）選擇。一台電腦可以有一個小容量的高速 SRAM，再加上大容量的 DRAM。一個可能的實作會是，將處理器位址空間的某塊區域劃分來容納 SRAM，剩下的則給 DRAM。作業系統的任務就會是最佳化地分配資料以善用 SRAM。基本上，在這種情境下，SRAM 是作為處理器的暫存器集的擴充來使用的。

雖然這是可能的實作，但並不可行。忽略將 SRAM 記憶體的實體資源映射到處理器的虛擬（virtual）位址空間的問題（這本身就非常難），這個方法會需要令每個行程（process）在軟體上管理記憶體區域的分配。記憶體區域的大小因處理器而異（也就是說，處理器有著不同容量的昂貴 SRAM 記憶體）。組成一支程式的每個模組都會要求它的那份快速記憶體，這會由於同步的需求而引入額外的成本。簡而言之，擁有快速記憶體的獲益將會完全被管理資源的間接成本（overhead）給吃掉。

所以，並非將 SRAM 置於 OS 或者使用者的控制之下，而是讓它變成由處理器透明地使用與管理的資源。在這種方式下，SRAM 是用以產生主記憶體中可能不久就會被處理器用到的資料的暫時副本。因為程式碼與資料具有時間（temporal）與空間局部性（spatial locality）。這表示，在短時間內，很可能會重複用到同樣的程式碼或資料。對程式碼來說，這表示非常有可能會在程式碼中循環（loop），使得相同的程式碼一次又一次地執行（*空間局部性*的完美例子）。資料存取在理想上也會被限定在一小塊區域中。即使在短時間內用到的記憶體並非鄰近，同樣的資料也有很高的機會在不久後再次用到（*時間局部性*）。對程式碼來說，代表––舉例來說––在一輪迴圈中會產生一次函式呼叫（function call），這個函式在記憶體中可能很遠，但呼叫這個函式在時間上則會很接近。對資料來說，代表一次使用的記憶體總量（工作集大小）理想上是有限的，但使用的記憶體––由於 RAM *隨機*存取的本質––並不是相鄰的。理解局部性的存在是 CPU 快取概念的關鍵，因為我們至今仍在使用它們。

一個簡單的計算就能看出快取在理論上有多有效。假設存取主記憶體花費 200 個週期，而存取快取記憶體花費 15 個週期。接著，程式使用 100 個資料元素各 100 次，若是沒有快取，將會在記憶體操作上耗費 2,000,000 個循環，而若是所有資料都被快取過，只要 168,500 個週期。提升了 91.5%。

用作快取的 SRAM 大小比起主記憶體小了好幾倍。根據作者使用具有 CPU 快取的工作站（workstation）的經驗，快取的大小總是主記憶體大小的 1/1000 左右（現今：4MB 快取與 4GB 主記憶體）。單是如此並不會形成問題。假如工作集（正在處理的資料集）的大小比快取大小還小，這無傷大雅。但是電腦不會無故擁有大量的主記憶體。工作集必定會比快取還大。尤其是執行多個行程的系統，其工作集的大小為所有個別的處理器與系統核心的大小總和。

應對快取的大小限制所需要的是，一組能在任何給定的時間點決定什麼該快取的策略。由於並非所有工作集的資料都會*正好*在相同的時間點使用，所以我們可以使用一些技術來暫時地將一些快取中的資料替換成別的資料。而且這也許能在真的需要資料之前就搞定。這種預取會去除一些存取主記憶體的成本，因為對程式的執行而言，這是非同步進行的。這所有的技術都能用來讓快取看起來比實際上還大。我們將會在 3.3 節討論它們。一旦探究完這全部的技術，協助處理器就是程式設計師的責任了。這些作法將會在第六節中討論。
