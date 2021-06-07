---
Keywords: DBMS, TPCC, C++
Copyright: (C) 2021 Riki Otaki
---

# TPC-Cを実装した

トランザクションエンジンの性能を測定するためのベンチマークである, [TPC-C](http://www.tpc.org/tpcc/)をC++で実装した. 名前はtpcc-runner.

https://github.com/wattlebirdaz/tpcc-runner

これによりC++で書かれたin-memoryのトランザクションエンジンであれば, その性能をいくつかのインターフェイスを実装することによって計測することができるようになった.

## TPC-Cとは何か

[TPC](http://tpc.org/default5.asp)とは, Transaction processing Performance Counsilの略で, トランザクションの処理性能を評価する組織である.
TPCはトランザクションの性能測定のやり方にいくつかのレギュレーションを設けることによって, 様々なトランザクションエンジンのパフォーマンスを統一した方法で計測しようとしている.
今回は中でも, OLTP(Online Transaction Processing)に特化した規定である, [TPC-C](http://www.tpc.org/tpcc/)を実装した.

## 具体的な内容

TPC-Cの内容を大まかに説明すると, 以下のようになる.

1. まずWarehouse, Item, Stock, District, Customer, History Order, NewOrder, OrderLineというレコードが入った9つのテーブルを作る. 各レコードの個数や, どのようなカラムがあるかは仕様に記述されている.
2. それらのテーブルに対して, 5つのトランザクション(それぞれNewOrder, Payment, Delivery, OrderStatus, StockLevelという名前がついている)を実行する. これらのトランザクションは, 1で作ったテーブルに対してreadやwriteを行う. また, 計測時のそれぞれのトランザクションの割合は仕様により定まっている.
3. トランザクションが1秒あたり何回commitされたかを計測し, Throughputを算出する.

TPC-Cはデータベースのアカデミックベンチマークとして, [YCSB](https://github.com/brianfrankcooper/YCSB)と共によく使われている. Concurrency Control関連の論文を読めば, そのほとんどでTPC-Cを用いてパフォーマンスが計測されていることがわかる.

## 実装の記録

どのように実装していったか簡単にメモしておく.

1. まず, 各レコードを仕様通りに定義した. 
2. その後各レコードを保存するためのデータベースを用意した. とりあえずシングルスレッドで動かすことを目標としていたので, 各テーブルは`std::map`で実装した. ただし, Historyレコードに関しては, Primary Keyをもたないので`std::deque`を使った. 複合キーは, 共用体を使うとうまく実装できた. 
3. データベースを初期化するためのInitializerを実装した. この際, uniform random intを生成する必要があったが, `std::uniform_int_distribution<int>`では遅すぎたので, Xoshiro256plusplusを用いて擬似乱数を生成することにした.
4. ここで, 一旦テーブルがちゃんとロードされているか, テストを書いた.
5. 5つのトランザクションを実装し始めた. トランザクションの中には, セカンダリテーブルを必要とするものがあったのでそれを実装した. また, 仕様で定義されているabort(NewOrderのうちの1%など)はusr abort, それ以外のconcurrency control関連のabortは sys abortと分類し, interfaceを実装した.
6. 各トランザクションは, DBへの変更をまずWriteSetにためて, precommit時に共有メモリに変更を加えるようにした. そうすることでabortを楽に実装できるようにした.
7. ここまでやるとパフォーマンスが計測できるようになっていた. パフォーマンスを計測すると, あまりよろしくなかった(ノートPCで18000 txns/sくらいだった)のでperfで解析すると, レコードのreadやwrite時にcopyしているのが重いことがわかった. copyを少なくするために, heapにメモリをallocateし, 使い回せるようにした.
8. さらにパフォーマンスを上げるためにcacheの実装, mallocではなくmimallocを使用などのチューニングをすると, 実行速度がノートPCで40000 txns/s, デスクトップで65000 txns/sくらいになっていた.

## 今後の展望

TPC-Cの実装に必要な最適化や高速化をしつつ, マルチスレッドでスケールするようなCCプロトコルを一つ実装し, そのパフォーマンスを計測したいと思っている.

## その他

レポジトリに実行の方法やパフォーマンスの見方などが書いてある.

https://github.com/wattlebirdaz/tpcc-runner

## 最後に

tpcc-runnerは[サイボウズ社のラボユース](https://labs.cybozu.co.jp/youth/requirements.html)という活動の一環として開発されました.
メンターである星野さんには頻繁にお時間をとっていただき, コードレビューや実装の方針を相談させていただきました.
心より感謝いたします.
