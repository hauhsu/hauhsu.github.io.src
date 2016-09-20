Title: Scala 的簡單檔案讀寫練習
Date: 2016-09-20
Modified: 2016-09-20
Category: Note
Tags: scala
Slug: scala-simple-file-read-write 
Author: Hau Hsu
Status: published


初學 scala，覺得困難。
沒想到只是要寫個簡單的程式（讓使用者輸入一個檔名，把那個檔案內容再寫到另一個檔案），
居然要搞半個小時。

``` scala 
import java.io
import scala.io.Source

object Demo {

   def printToFile(f: java.io.File)(op: java.io.PrintWriter => Unit) {
     val p = new java.io.PrintWriter(f)
     try { op(p) } finally { p.close() }
   }

   def main(args: Array[String]) {
      print("Please enter input file name: " )
      val infile = Console.readLine
      println("Thanks, you just typed: " + infile)

      val outfile = new java.io.File("outfile.txt")

      printToFile(outfile) {
        p => Source.fromFile(infile).foreach {
          p.print
        }
      }
   }
}
```

Demo 是一個 object，裡面有兩個函式：`printToFile`和`main`。  
`printToFIle` 是從 [stackoverflow](http://stackoverflow.com/questions/4604237/how-to-write-to-a-file-in-scala) 上的答案抄下來的。這個函式有兩組參數：要寫的檔案和定義你要怎麼寫函式。`op`是一個函式，會吃一個 java.io.PrintWrriter 的參數，而返回值是 Unit。

在 main 裡，我們先用 `Console.readLine` 讓使用者輸入要讀的檔案名稱，將這個名稱存到 `infile` 變數中。接著用 `java.io.File` 開啟 outfile.txt。
接著呼叫 `printToFile` 函式來將 `infile` 中的資料寫到 `outfile`。這邊呼叫的方式有點特別，當某個函式有兩組輸入參數時，scala 允許你用大括號來把第二組參數包起來。這樣如果第二組參數是一個函式時程式會比較好讀。這邊就是一個例子，`printToFile`的第二個參數是一個函式，而      

``` scala
    printToFile(outfile) {
      p => Source.fromFile(infile).foreach {
        p.print
      }
    }
```
其實是

``` scala 
   printToFile(outfile) (
     p => Source.fromFile(infile).foreach {
       p.print
     }
   )
```
`p` 是一個 `java.io.PrintWriter`，這個 PrintWriter 是在 `printToFile` 裡面才被實體化的，在這邊是用 p 來代表這個傳入的 `PrintWriter`。`=>`後面就是這個函數做的事情：用 Source.fromFile 打開 infile，然後將檔案裡的每一行用 `p` 的 `print` 函式處理。

編譯和執行
----------
把上面的程式碼存成`fileio.scala`，然後用以下指令編譯和執行：  
`$ scalac fileio.scala`  
`$ echo "aaa\nbbb\nccc\n" > test.txt  #隨便放一些字到test.txt中`   
`$ scala Demo`  

輸出：

``` 
Please enter input file name: test.txt
Thanks, you just typed: test.txt
```

為什麼好像有些基本的功能 scala 都沒有而必須用 java 的函式庫？像 scala 可以讀檔案，但是寫檔案要用 java 的 io ...。感覺沒有學 java 就跳進 scala 有點吃力QQ。
