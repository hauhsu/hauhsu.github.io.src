Title: GDB TUI - 原來 GDB 要這樣用
Date: 2018-03-15
Tags: gdb
Slug: gdb-tui
Author: Hsu Hau
Status: published
Summary:  GDB 的文字使用者介面介紹和教學

一直以來我都覺得 GDB 世界難用，要記一堆指令，要看程式跑到哪裡只能用 list
，還一次只秀個十行給你。我也不知道我在哪個檔案的哪一行設了 break point 等等。
腦袋要記和要想像的東西太多，每次都用一下子就覺得頭昏腦脹，最後又回到 printf
debug 大法，頂多就是遇到 segmentation fault 很快的看能不能用 GDB 找到死在哪。

直到我打開了 TUI，整個世界都不一樣了！！

以下會用一個有問題的小程式來 demo 用 TUI debug 的過程。

首先準備以下程式並且命名為 `sum.c`：


``` c
#include <stdio.h>

int sum(int nums[], int size) {
  int i, sum;
  for (i = 0, sum = 0; i <= size; i++) {
    sum += nums[i];
  }
  return sum;
}

int main(int argc, char *argv[])
{
  int nums[] = {1, 2, 3, 4, 5};
  int s = sum(nums, 5);
  printf("sum = %d\n", s);

  return 0;
}
```

這隻程式裡有一個函式，給他一個整數的陣列和陣列大小，他會算出這個陣列中所有數字的總和並回傳。在 main 當中則是呼叫這個函式去算 1~5 的總和然後印出來。

編譯後執行，沒想到這麼簡單的程式竟然印出奇怪的數字？！

~~~
$ gcc sum.c -o sum -g
$ ./sum
sum = 32782
~~~

這是怎麼一回事呢？我們加上 debug symbol 後重新編譯，然後打開 gdb 並且進入 TUI 模式：

~~~
$ gcc sum -o sum -g
$ gdb sum -tui
~~~


或者也可以用一般模式進入 gdb 以後再用 Ctrl-x Ctrl-a 打開 TUI。

![]({filename}/images/gdb-tui/gdb1.png)

開了以後視窗會分成上下兩半，上半部是目前在執行的程式碼， 下半部就是 GDB 輸入指令的介面。進入 TUI 模式後，方向鍵的上下變成移動上視窗程式碼的部分。如果要像一般 GDB 執行前一個或下一個指令的話，可以用 Ctrl-p 和 Ctrl-n。

接著我們在 main 一開始的時候設一個中斷點，再開始執行程式，這樣程式就會停在 main 的第一行：

~~~
(gdb) b main
(gdb) r
~~~

下圖可以看到程式正在執行的地方會被反白起來，然後在行數的左邊會顯示中斷點。

![]({filename}/images/gdb-tui/gdb2.png)

接著我想知道每次執行迴圈後 sum 的值怎麼改變，所以我們在第六行設一個中斷點，並且用 `command` 來設定每次到這個中斷點都要把 i 、要加的數目、以及加之前的 sum 印出來：

~~~
(gdb) b 6
(gdb) command
Type commands for breakpoint(s) 2, one per line.
End with a line saying just "end".
>p i
>p nums[i]
>p sum
>end
(gdb) c
~~~

![]({filename}/images/gdb-tui/gdb3.png)

現在 `i` 跟 `sum` 都是 0，準備要加的數字是 1，看起來沒問題，接著用 `continue` 往下執行一次迴圈：

~~~
(gdb) c
~~~

![]({filename}/images/gdb-tui/gdb4.png)

現在 `i` 跟 `sum` 都是 1，要注意 sum
的值是上一輪算完的結果。如此再重複四次，我們可以直接按 <Enter> 重複上一個指令。

![]({filename}/images/gdb-tui/gdb5.png)

到這邊應該要算完了，`i` 的值是 4，要加的數字是 5，加上去之前的 `sum` 是十，看起來也沒錯，到底錯在哪裡？我們按下 <Enter> ，神奇的事情出現了：

![]({filename}/images/gdb-tui/gdb6.png)

原來在設定 for 迴圈離開條件的地方寫成了 `i <= size`，會讓迴圈多跑一次，結果明明我們給的陣列只有五個元素，這個函式卻會去算到第六個元素造成錯誤。


有了視覺化的幫助真的讓 GDB 好用非常非常多，程式執行到什麼檔案的哪一行、break point 設在哪裡一目瞭然，這也幫我解了工作上的第一個 issue :))。
