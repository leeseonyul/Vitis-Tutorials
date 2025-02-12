<p align="right"><a href="../../../../../README.md">English</a> | <a>日本語</a></p>

# アプリケーションのプロファイリング

Vitis は、コンパイル時にさまざまなシステムおよびカーネル リソースのパフォーマンス レポートを生成します。また、XRT では、ソフトウェアおよびハードウェア エミュレーションとシステム実行モードの両方で、アプリケーションの実行中にプロファイリング データを収集します。コンパイルおよび実行からリンクされた多数のレポートをリンクした実行サマリ レポートは、アクティブ ビルドの実行後に自動的に生成され、Vitis アナライザーで表示できます。

プロファイリング レポートおよびデータは、アプリケーションのパフォーマンスのボトルネックを特定し、システム内で問題を検出し、デザインを最適化してパフォーマンスを向上するために使用できます。プロファイリング レポートを取得するには、関連するオプションが xrt.ini ファイルでオンになっていることを確認します。このファイルは、ホスト実行ファイルと同じディレクトリに配置する必要があります。このチュートリアルでは、提供されている既に処理済みの xrt.ini を使用します。./sw/duild/ ディレクトリを確認してください。詳細は、Vitis オンライン資料の [xrt.ini ファイル](https://japan.xilinx.com/html_docs/xilinx2020_1/vitis_doc/obl1532064985142.html)を参照してください。

```
[Debug]
profile=true
timeline_trace=true
data_transfer_trace=fine
```

ここでは、U200 カードの rtc\_alpha\_tb のハードウェア実行結果を例に使用します。実行サマリには、6 つのカテゴリのレポートがリストされます。これは、システム実行からの実行サマリであるため、波形は使用できません。

```
vitis_analyzer rtc_alpha_hw.xclbin.run_summary
```

## 実行ガイダンス

ガイダンスには、レポートされた違反に関するメッセージ、推奨される解決方法の概要、詳細な解決方法へのウェブ リンクが含まれます。このルールは、広範なリファレンス デザインのセットに基づいた汎用ルールであるため、すべてのデザインに適用されるとは限りません。そのため、ガイダンス ルールを理解して、特定のアルゴリズムおよび要件に基づいて適切な操作を実行するようにしてください。

<div align="center"><img src="./images/hw_guidance.PNG" alt="ハードウェア実行ガイダンス" ></div>

## プラットフォームとシステム ダイアグラム

プラットフォームとシステム ダイアグラムには、プラットフォーム リソースとプラットフォームに統合されたカーネル コードが表示されます。システム ダイアグラムは、XCLBIN で使用されるメモリ バンクまたは PLRAM、および CU の関数引数が AXI4 インターフェイスにどのように接続されているかを示します。これには、実行時のプロファイル データが含まれます。また、下の表のリソース情報は、システム ダイアグラムの各カーネルまたは CU の横にあるボックスに表示することもできます。

右上の **\[Settings]** ボタンを使用すると、未使用のメモリ、インターフェイス ポート、プロファイル情報、およびリソース情報を表示/非表示を切り替えることができます。

<div align="center"><img src="./images/hw_sys_diagram.PNG" alt="ハードウェア システム ダイアグラム" ></div>

## プロファイル サマリ

カーネルとホスト間のトラフィックのプロファイル データの取り込みを有効にすると、追加のリソースが消費され、パフォーマンスに影響する可能性があります。そのため、このチュートリアルのプリビルド済みの XCLBIN ファイルからは、これらの要素をソースで削除してあります。

カーネルにアクセラレーション モニターと AXI パフォーマンス モニターを追加すると、Vitis コンパイラのリンク プロセスで `--profile_kernel` オプションを追加して、カーネルのデータ プロファイリングを有効にできます。例として、次の ./hw/Makefile への次の変更を参照してください。詳細は、Vitis オンライン資料の [Vitis コンパイラ コマンド](https://japan.xilinx.com/html_docs/xilinx2020_1/vitis_doc/vitiscommandcompiler.html#wrj1504034328013)を参照してください。

```
rtc_alpha_$(TARGET).xclbin: $(XOS_RTC_ALPHA)
	v++ -l $(XOCCFLAGS) $(XOCCLFLAGS) --config xclbin_rtc_alpha.ini --profile_kernel data:all:all:all  -o $@ $(XOS_RTC_ALPHA)

rtc_gen_test_$(TARGET).xclbin: $(XOS_RTC_GEN_TEST)
	v++ -l $(XOCCFLAGS) $(XOCCLFLAGS) --config xclbin_rtc_gen_test.ini --profile_kernel data:all:all:all  -o $@ $(XOS_RTC_GEN_TEST)
```

<div align="center"><img src="./images/hw_profile_summary.PNG" alt="プロファイル サマリ" ></div>
XCLBIN ファイルの生成時に `--profile_kernel` オプションを追加しなくても、ほとんどのプロファイリング レポートは利用できます。カーネル データ転送のような一部のセクションだけ、データが表示されません。

<div align="center"><img src="./images/no_kernel_data.PNG" alt="カーネル データなし" ></div>

## アプリケーション タイムライン

アプリケーション タイムラインは、ホストとカーネルのイベント情報を収集し、共通のタイムラインに表示します。これは、システムの全体的な状態とパフォーマンスを視覚的に表示して理解するのに役立ちます。このグラフィカルな表示により、カーネル同期および並列実行の効率に関する問題を発見しやすくなります。

<div align="center"><img src="./images/hw_timeline.PNG" alt="アプリケーション タイムライン" ></div>

プロファイリングの詳細および Vitis アナライザーの使用方法については、Vitis オンライン資料の[アプリケーションのプロファイリング](https://japan.xilinx.com/html_docs/xilinx2020_1/vitis_doc/profilingapplication.html)を参照してください。


---------------------------------------


<p align="center"><sup>Copyright&copy; 2020 Xilinx</sup></p>
<p align= center class="sphinxhide"><b><a href="../../../../README.md">メイン ページに戻る</a> &mdash; <a href="../../../README.md">ハードウェア アクセラレータ チュートリアルの初めに戻る</a></b></p></br>
<p align="center"><sup>この資料は 2021 年 2 月 8 日時点の表記バージョンの英語版を翻訳したもので、内容に相違が生じる場合には原文を優先します。資料によっては英語版の更新に対応していないものがあります。
日本語版は参考用としてご使用の上、最新情報につきましては、必ず最新英語版をご参照ください。</sup></p>
