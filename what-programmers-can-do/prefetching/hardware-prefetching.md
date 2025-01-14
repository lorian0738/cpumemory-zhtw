# 6.3.1. 硬體預取

CPU 啟動硬體預取的觸發，通常是二或多個快取錯失的某種模式的序列。這些快取錯失可能在快取行之前或之後。在舊的實作中，只有鄰近快取行的快取錯失會被識別出來。使用當代硬體，步伐也會被識別出來，代表跳過固定數量的快取行會被識別為一種模式並被適當地處理。

若每次單一的快取錯失都會觸發一次硬體預取，對於效能來說大概很糟。隨機記憶體存取模式 –– 例如存取全域變數 –– 是非常常見的，而產生的預取會大大地浪費 FSB 頻寬。這即是為何啟動預取需要至少兩次快取錯失。處理器現今全都預期有多於一條記憶體存取的串流。處理器試著自動將每個快取錯失指派給這樣的一條串流，並且在達到門檻時啟動硬體預取。CPU 現今能追蹤更高階快取的八到十六條單獨的串流。

負責模式識別的單元與各自的快取相關聯。可以有一個 L1d 與 L1i 快取的預取單元。很可能有一個 L2 與更高階快取的預取單元。L2 與更高階快取的預取單元是被所有使用相同快取的其它處理器核與 HT 所共享。八到十六條單獨串流預取單元的數量便因而迅速減少。

預取有個大弱點：它無法跨越分頁邊界。理解到 CPU 支援需求分頁（demand paging）時，原因應該很明顯。若是預取被允許橫跨分頁邊界，存取可能會觸發一個事件，以令分頁能夠被取得。這本身可能很糟，尤其是對效能而言。更糟的是預取器並不知道程式或作業系統本身的語義（semantic）。它可能因此預取實際上永遠不會被請求的分頁。如此意味著預取器會運行超過處理器曾以可識別模式存取過的記憶體區域盡頭。這不只可能，而且非常有可能。若是處理器 –– 作為一次預取的一個副作用 –– 觸發對這樣的分頁的請求，作業系統甚至可能會在這種請求永遠也不會發生時完全扔掉它的追蹤紀錄。

因此重要的是認識到，無論預取器在預測模式上有多厲害，程式也會在分頁邊界上歷經快取錯失，除非它明確地從新的分頁預取或是讀取。這是如 6.2 節描述的最佳化資料佈局、以藉由將不相關的資料排除在外來最小化快取污染的另一個理由。

由於這個分頁限制，處理器現今並沒有非常複雜的邏輯來識別預取模式。以仍佔主導地位的 4k 分頁大小而言，有意義的也就這麼多。這些年來已經提高識別步伐的位址範圍，但超過現今經常使用的 512 位元組窗格（window）可能沒太大意義。目前的預取單元並不認得非線性的存取模式。這種模式較有可能是真的隨機、或者至少足夠不重複到令試著識別它們不具意義。

若是硬體預取被意外地觸發，能做的只有這麼多。一個可能是試著找出這個問題，並稍微改變資料與／或程式佈局。這大概滿困難的。可能有特殊的在地化（localized）解法，像是在 x86 與 x86-64 處理器上使用 `ud2` 指令[^35]。這個無法自己執行的指令是在一條間接的跳躍指令後被使用；它被作為指令獲取器（fetcher）的一個信號使用，表示處理器不應浪費精力解碼接下來的記憶體，因為執行將會在一個不同的位置繼續。不過，這是個非常特殊的情況。在大部分情況下，必須要忍受這個問題。

能夠完全或部分地停用整個處理器的硬體預取。在 Intel 處理器上，一個特定模型暫存器（Model Specific Register，MSR）便用於此（IA32\_MISC\_ENABLE，在許多處理器上為位元 9；位元 19 只停用鄰近快取行預取）。這在大多情況下必須發生在系統核心中，因為它是個特權操作。若是數據分析顯示，執行於系統上的一個重要的應用程式因硬體快取而遭受頻寬耗竭與過早的快取逐出，使用這個 MSR 是一種可能性。



[^35]: 或是 non-instruction。這是推薦的未定義操作碼。

