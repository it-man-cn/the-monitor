# How to monitor Microsoft Azure VMs

> *This post is part 1 of a 3-part series on monitoring Azure virtual machines. [Part 2](/blog/how-to-collect-azure-metrics) is about collecting Azure VM metrics, and [Part 3](/blog/monitor-azure-vms-using-datadog) details how to monitor Azure VMs with Datadog.*

*このポストは、"Azureの監視"3回シリーズのPart 1です。 Part 2は、[「Azureの仮想マシンメトリクスの収集」](/blog/how-to-collect-azure-metrics)で、Part 3は、[「Datadogを使ったAzuzre仮想マシンの監視」](/blog/monitor-azure-vms-using-datadog)になります。*

## What is Azure?

> Microsoft Azure is a cloud provider offering a variety of compute, storage, and application services. Azure services include platform-as-a-service (PaaS), akin to Google App Engine or Heroku, and infrastructure-as-a-service (IaaS). In the most recent Gartner “Magic Quadrant” rating of cloud IaaS providers, Azure was one of only two vendors (along with Amazon Web Services) to place in the “Leaders” category.

マイクロソフト Azureは、さまざまな計算用インスタンス、ストレージ、およびアプリケーションサービスを提供しているクラウドプロバイダーです。Azureのサービスには、Google App EngineやHerokuに代表されるようなPaaS(プラットホーム・アズ・サービス)と、IaaS(インフラストラクチャー・アズ・サービス)があります。Azureは、ガートナー社が公開しているIaaSプロバイダーの“Magic Quadrant”の直近の指標で、Amazon Web Serviceと並んで、"リーダー"カテゴリーに
ランク付けされました。

> In this article, we focus on IaaS. In an IaaS deployment, Azure’s basic unit of compute resources is the virtual machine. Azure users can spin up general-purpose Windows or Linux (Ubuntu) VMs, as well as machine images for applications such as SQL Server or Oracle.

このポストでは、AzureのIaaSにフォーカスし解説していきます。AzureのIaaSサービスでは、計算リソースの基本は、仮想マシンです。Azureのユーザは、汎用目的のWindowsが動作している仮想マシンや、Linux(Ubuntu)が動作している仮想マシンを起動したり、SQL ServerやOracleなどのアプリケーションがインストールされた仮想マシンイメージも起動することができます。

## Key Azure metrics

> Whether you run Linux or Windows on Azure, you will want to monitor certain basic VM-level metrics to make sure that your servers and services are healthy. Four of the most generally relevant metric types are **CPU usage**, **disk I/O**, **memory utilization** and **network traffic**. Below we’ll briefly explore each of those metrics and explain how they can be accessed in Azure.

Azureの上でLinux又はWindowsを実行した場合、サーバやサービスの適切な運用を確保するために、仮想マシンレベルで基本メトリクスを監視したいはずです。その際、最も関連性のあるは、**CPU usage**、 **disk I/O**、**memory utilization**、**network traffic** の４つのメトリクスになります。下記では、これらの各メトリクスを手短に紹介し、Azure上でどのようにアクセスすることができるかを解説していきます。

> This article references metric terminology [introduced in our Monitoring 101 series](/blog/monitoring-101-collecting-data/), which provides a framework for metric collection and alerting.

このポストでは、「メトリクスの収集方法やアラートの設定方法に関する基礎的な知識」のポストである[introduced in our Monitoring 101 series][10]で紹介したメトリクス用語を使っています。

> Azure users can monitor the following metrics using [the Azure web portal](https://portal.azure.com/) or can access the raw data directly via the Azure diagnostics extension. Details on how to collect these metrics are available in [the companion post](/blog/how-to-collect-azure-metrics) on Azure metrics collection.

Azureのユーザーは、以下に紹介するメトリクスを[AzureのWebポータル](https://portal.azure.com/)上から監視することがでます。更に、”Azure diagnostics extension”を介し、直接生のデータにアクセスすることもできます。Azureからメトリクスを収集する詳しい手順に関しては、このシリーズのPart2[「Azureの仮想マシンメトリクスの収集」](/blog/how-to-collect-azure-metrics)の解説を参照してください。

### CPU metrics

> CPU usage is one of the most commonly monitored host-level metrics. Whenever an application’s performance starts to slide, one of the first metrics an operations engineer will usually check is the CPU usage on the machines running that application.

CPUの使用率は、最も一般的に監視対象となるホストレベルのメトリクスの一つです。アプリケーションの性能が低下し始めた際、オペレーションエンジニアが最初にチェックするメトリクスの一つが、通常そのアプリケーションを実行しているマシンのCPU使用率になります。

| **Name**            | **Description**                       | **[Metric type](/blog/monitoring-101-collecting-data/)** |
|---|----|-----|
| CPU percentage      | Percentage of time CPU utilized       | Resource: Utilization                                    |
| CPU user time       | Percentage of time CPU in user mode   | Resource: Utilization                                    |
| CPU privileged time | Percentage of time CPU in kernel mode | Resource: Utilization                                    |

> CPU metrics allow you to determine not only how utilized your processors are (via **CPU percentage**) but also how much of that utilization is accounted for by user applications. The **CPU user time** metric tells you how much time the processor spent in the restricted “user” mode, in which applications run, as opposed to the privileged kernel mode, in which the processor has direct access to the system’s hardware. The **CPU privileged time** metric captures the latter portion of CPU activity.

CPUメトリクスを監視することによりプロセッサーの利用状況を(**CPU percentage** を介し)把握できるだけはなく、各ユーザーのアプリケーションがどの程度CPUを占有しているかを把握することができるようになります。**CPU user time** メトリクスからは、CPUが"restricted 'user' mode"で、アプリケーションを実行している時間を知ることができます。その反対に、**CPU privileged time** メトリクスからは、CPUが"privileged kernel mode"で、システムのハードウェアに直接アクセスしている時間を知ることができます。

#### Metric to alert on: CPU percentage

> Although a system in good health can run with consistently high CPU utilization, you will want to be notified if your hosts’ CPUs are nearing saturation.

システムが良好な状態にあってもCPUの使用率が常に高い場合、ホストの個々のCPUが飽和状態に近づいた際には、通知されるようにしたいものです。

[![Azure CPU heat map](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/azure-1-cpu.png)](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/azure-1-cpu.png)

### Disk I/O metrics

> Monitoring disk I/O is critical for understanding how your applications are impacting your hardware, and vice versa. For additional visibility beyond the VM-level metrics covered here, you can also collect metrics from your Azure storage accounts to determine if your storage is being throttled or has availability issues that could impact performance.

ディスクI/Oの監視は、アプリケーションがハードウェアに及ぼしている影響を理解するのには非常に役に立ちます。仮想マシンレベルとして紹介したメトリクスの情報に加えAzureアカウントのスレージメトリクスを収集することで、そのストレージが制限を受けているか、又パフォーマンスの影響を与える可能性のある可用性の問題を抱えているかを、把握できるようになります。

| **Name**   | **Description**                  | **[Metric type](/blog/monitoring-101-collecting-data/)** |
|---|----|-----|
| Disk read  | Data read from disk, per second  | Resource: Utilization                                    |
| Disk write | Data written to disk, per second | Resource: Utilization                                    |

#### Metric to alert on: Disk read

Monitoring the amount of data read from disk can help you understand your application’s dependence on disk. If the application is reading from disk more often than expected, you may want to add a caching layer or switch to faster disks to relieve any bottlenecks.

ディスクから読み込まれるデータ量を監視することは、アプリケーションのディスクへの依存度を知るのに役立ちます。もしもアプリケーションが予想していたよりも頻繁にディスクの読み込みをしている場合は、ボトルネック(遅延)の可能性を解消するために、キャッシング層を追加したり、高速なディスクを選択することになります。

#### Metric to alert on: Disk write

> Monitoring the amount of data written to disk can help you identify bottlenecks caused by I/O. If you are running a write-heavy application, you may wish to upgrade[the size of your VM](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-size-specs/) to increase the maximum number of IOPS (input/output operations per second).

ディスクへ書き込むデータ量を監視することは、ディスクI/Oに起因するボトルネック(遅延)を発見するのに役立ちます。ディスクへの書き込みの多いアプリケーションを実行している場合、IOPSの上限値を増やすために[仮想マシンのサイズ](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-size-specs/)をアップグレードする必要があるかもしれません。

[![Azure disk write speed](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-disk-write-2.png)](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-disk-write-2.png)

### Memory metrics

> Monitoring memory usage can help identify low-memory conditions and performance bottlenecks.

メモリの使用量を監視することは、メモリの残量が少なくなった場合のパフォーマンスボトルネックを発見するのに役立ちます。

| **Name**         | **Description**                                               | **[Metric type](/blog/monitoring-101-collecting-data/)** |
|---|----|-----|
| Memory available | Free memory, in bytes/MB/GB                                   | Resource: Utilization                                    |
| Memory pages     | Number of pages written to or retrieved from disk, per second | Resource: Saturation                                     |

#### Metric to alert on: Memory pages

> Paging events occur when a program requests a [page](https://en.wikipedia.org/wiki/Page_(computer_memory)) that is not available in memory and must be retrieved from disk, or when a page is written to disk to free up working memory. Excessive paging can introduce slowdowns in an application. A low level of paging can occur even when the VM is underutilized—for instance, when the virtual memory manager automatically trims a process’s [working set](https://msdn.microsoft.com/en-us/library/windows/desktop/cc441804(v=vs.85).aspx) to maintain free memory. But a sudden spike in paging can indicate that the VM needs more memory to operate efficiently.

プログラムがメモリ内にページを確保できずにディスクからその領域を確保している場合、利用可能なメモリ領域を確保するためにディスクにページを退避する場合に、ページングイベントが発生します。過度に発生するページングは、アプリケーションの実行速度を低下の原因になります。仮想マシンの負荷が低くても、多少のページングは発生しています。例えば、”virtual memory manager”(仮想メモリーのマネージャー)
が、[プロセスが使っているメモリ](https://msdn.microsoft.com/en-us/library/windows/desktop/cc441804(v=vs.85).aspx)を整理し、空きメモリのスペースを確保する場合です。しかし、突然のページング量の増加は、その仮想マシンが効率的に動作するためにメモリを追加する必要があることを教えてくれています。

 [![Azure memory paging](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-memory-pages.png)](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-memory-pages.png)

### Network metrics

> Azure’s default metric set provides data on network traffic in and out of a VM. Depending on your OS, the network metrics may be available in bytes per second or via the number of TCP segments sent and received. Because TCP segments are limited in size to 536 bytes each, the number of segments sent and received provides a reasonable proxy for the overall volume of network traffic.

Azurreが提供するデフォルトのメトリクスセットには、仮想環境が使用しているネットワークの通信量データも提供されています。Azureのネットワークメトリクスでは、使っているOS応じ、送受信しているデータの"バイト/秒"や"TCPのセグメント数"で、通信量が提供されます。TCPのセグメン数は、536バイト毎に計算されるので、送受信したセグメントの数値はネットワークの全体的な通信量を監視するためのメトリクスとして使うことができます。

| **Name**              | **Description**                  | **[Metric type](/blog/monitoring-101-collecting-data/)** | **Availability** |
|---|----|-----|-----|
| Bytes transmitted     | Bytes sent, per second           | Resource: Utilization                                    | Linux VMs        |
| Bytes received        | Bytes received, per second       | Resource: Utilization                                    | Linux VMs        |
| TCP segments sent     | Segments sent, per second        | Resource: Utilization                                    | Windows VMs      |
| TCP segments received | Data written to disk, per second | Resource: Utilization                                    | Windows VMs      |

#### Metric to alert on: Bytes/TCP segments sent

> You may wish to generate [a low-urgency alert](/blog/monitoring-101-alerting/#low) when your network traffic nears saturation. Such an alert may not notify anyone directly but will record the event in your monitoring system in case it becomes useful for investigating a performance issue.

ネットワークの通信量が飽和状態になりかけている時、[通信速度が低下している旨の緊急アラート(a low-urgency alert)](/blog/monitoring-101-alerting/#low)を発生させておきたいはずです。 この種のアラートは誰にも通知しません。しかし、パフォーマンスの問題が発生した際の有用な情報として監視システム内にイベントとして記録しておきます。

#### Metric to alert on: Bytes/TCP segments received

> If your network traffic suddenly plummets, your application or network may be overloaded.

ネットワークの通信量が、突然急降下(垂直に落ちる)した場合、アプリケーションかネットワークが過負荷状態にあることを示しています。

 [![Azure network out](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-network-out.png)](https://don08600y3gfm.cloudfront.net/ps3b/blog/images/2015-08-azure/1-network-out.png)

## Conclusion

> In this post we’ve explored several general-purpose metrics you should monitor to keep tabs on your Azure virtual machines. Monitoring the metric set listed below will give you a high-level view of your VMs’ health and performance:

このポストでは、Azureの仮想マシンの状況を把握しておくために監視しておくべき最も有用なメトリクスについて紹介してきました。以下のリストにあるメトリクスを監視することで、Azure上で起動している仮想マシンの健全性と稼働状態を把握できるようになるはずです:

-   [CPU percentage](#cpu-percentage)
-   [Disk read/write](#disk-read)
-   [Memory pages](#memory-pages)
-   [Network traffic sent/received](#network-sent)

> Over time you will recognize additional, specialized metrics that are relevant to your applications. [Part 2 of this series](/blog/how-to-collect-azure-metrics/) provides step-by-step instructions for collecting any metric you may need from Azure.

時間が経つにつれて、アプリケーションに関連する専門的なメトリクスが他にも存在していることに気づくでしょう。このシリーズのPart2 [「Azureの仮想マシンメトリクスの収集」](/blog/how-to-collect-azure-metrics/)では、Azureの仮想マシンからメトリクスを収集するために必要な手順を解説していくことにします。

## Acknowledgments

> Many thanks to reviewers from Microsoft for providing important additions and clarifications prior to publication.

このポストを公開するに当たり、記事を事前にレビューし、重要なフィードバックと細部にわたる解説を提供してくれたMicrosoftのレビューワーの皆さんに感謝します。

------------------------------------------------------------------------

> *Source Markdown for this post is available [on GitHub](https://github.com/DataDog/the-monitor/blob/master/azure/how_to_monitor_microsoft_azure_vms.md). Questions, corrections, additions, etc.? Please [let us know](https://github.com/DataDog/the-monitor/issues).*

*このポストのMarkdownソースは、[GitHub](https://github.com/DataDog/the-monitor/blob/master/azure/how_to_monitor_microsoft_azure_vms.md)で閲覧することができます。質問、訂正、追加、などがありましたら、[GitHubのissueページ](https://github.com/DataDog/the-monitor/issues)を使って連絡を頂けると幸いです。*
