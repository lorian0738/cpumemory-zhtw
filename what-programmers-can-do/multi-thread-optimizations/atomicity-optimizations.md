# 6.4.2. 原子性最佳化

假如多條執行緒同時修改了相同的記憶體位置，處理器並不保證任何具體的結果。這是個為了避免在所有情況的 99.999% 中的不必要成本而做出的慎重決定。舉例來說，若有個在「S」狀態的記憶體位置、並且有兩條執行緒同時必須增加它的值的時候，在從快取讀出舊值以執行加法之前，執行管線不必等待快取行變為「E」狀態。而是會讀取當前快取中的值，並且一旦快取行變為「E」狀態，新的值便會被寫回去。若是在兩條執行緒中的兩次快取讀取同時發生，結果並不如預期；其中一個加法會沒有效果。

對於可能發生並行操作的情況，處理器提供了原子操作。舉例來說，這些原子操作可能在直到能以像是原子地對記憶體位置進行加法的方式執行加法之前，不會讀取舊值。除了等待其它核心與處理器之外，某些處理器甚至會將特定位址的原子操作發給在主機板上的其它裝置。這全都會令原子操作變慢。

處理器廠商決定提供不同的一組原子操作。早期的 RISC 處理器，與代表簡化（reduced）的「R」相符，提供了非常少的原子操作，有時僅有一個原子的位元設置與測試。[^40]在光譜的另一端，我們有提供了大量原子操作的 x86 與 x86-64。普遍來說可用的原子操作能夠歸納成四類：

<dl>
  <dt>位元測試</dt>
  <dd>這些操作原子地設置或者清除一個位元，並回傳一個代表位元先前是否被設置的狀態。</dd>

  <dt>載入鎖定／條件儲存（Load Lock/Store Conditional，LL/SC）[^41]</dt>
  <dd>LL/SC 操作成對使用，其中特殊的載入指令用以開始一個事務（transaction），而最後的儲存僅會在這個位置沒有在這段期間內被修改的情況才會成功。儲存操作指出了成功或失敗，所以程式能夠在必要時重複它的工作。</dd>

  <dt>比較並交換（Compare-and-Swap，CAS）</dt>
  <dd>這是個三元（ternary）操作，僅在當前值與第三個參數值相同的時候，將一個以參數提供的值寫入到一個位址中（第二個參數）；</dd>

  <dt>原子算術</dt>
  <dd>這些操作僅在 x86 與 x86-64 可用，其能夠在記憶體位置上執行算術與邏輯操作。這些處理器擁有對這些操作的非原子版本的支援，但 RISC 架構則否。所以，怪不得它們的可用性是有限的。</dd>
</dl>



[^40]: HP Parisc 仍然沒有提供更多的操作...

[^41]: 有些人會使用「鏈結（linked）」而非「鎖定」，這是一樣的。
