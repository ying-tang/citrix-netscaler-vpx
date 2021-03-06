---
copyright:
  years: 1994, 2017
lastupdated: "2017-11-02"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Citrix NetScaler VPX を使用したグローバル・ロード・バランシング

グローバル・サーバー・ロード・バランシング (GSLB) は、一般的に異なる地理的位置にある複数のサーバー/インスタンスにトラフィックを分散させるメカニズムです。 これはグローバル・バランシング・エンジン/サーバーを持つという概念であり、このエンジン/サーバーはクライアントからのトラフィック要求を受信し、管理者によって選択および構成された基準/アルゴリズムを使って決められた特定の地域に、そのトラフィックをリダイレクトします。 これを実現するために、{{site.data.keyword.BluSoftlayer_notm}}ネットワーク内では、認識済みの 2 つの方法を使用できます。

* **CDN:** コンテンツ配信ネットワーク (CDN) は、コンテンツとリッチ・メディア (イメージやビデオなど) を配信するために発行され、最小の待ち時間と最高速度を維持しながら、地理的に分散されたノードにコンテンツを分散させます。 通常は、Web サイト/アプリケーション全体ではなく、コンテンツの特定の部分を分散させる必要がある場合に実装されます。 {{site.data.keyword.BluSoftlayer_notm}} はこのサービスを提供します。詳しくは、[こちら](https://console.bluemix.net/docs/infrastructure/CDN/getting-started.html#getting-started)を参照してください。 
* **NetScaler VPX:** 通常のローカル・ロード・バランシングと同様に、VPX は同様のオブジェクト階層を使用して、複数の地域間でトラフィックのロード・バランシングを行います。 NetScaler は、DNS ベースのグローバル・ルックアップを使用して、選択されたロケーション/サイトに対応するそれぞれのレコードを選択します。この選択は、管理者が事前構成した基準に基づいて行われます。 着信セクションは、このオプション/オファリングに展開されます。

コンテンツの分散には HTTP リダイレクトなど他の手法も利用可能であり、これも Citrix NetScaler VPX を使用して実装できます。 

## VPX での GSLB について

NetScaler VPX は、CLI/GUI で GSLB 機能をアクティブにするだけで、GSLB 機能に対応できます。 

GSLB は、ローカル・ロード・バランシング・デプロイメントで使用されるものとほぼ同様のコンポーネント/オブジェクトを使用します。この場合、エンティティーは階層モデルを使用して定義されます。

GSLB は、さまざまな目的で実装されることがあります。 以下のような目的が考えられます。

* **負荷の共有/分散:** 複数のロケーション間でリソースを効率的かつスマートに分散します。
* **災害時リカバリー:** メイン・ロケーション (1 次ロケーション) がダウンした場合や、サービスをレンダリングできない場合に使用されます。 このシナリオでは、代替/バックアップ・ロケーションへの引き継ぎが可能です。
* **パフォーマンス/シェーピング:** コンテンツを終端システムに近い位置に配置したり、問題のサービスを強化する方法で配置したりすることができます。

GSLB プロセス/デプロイメントの主要なコンポーネントとエンティティーは、以下のとおりです。

* **仮想サーバー:** VIP はクライアントが要求を送信する先の IP アドレスであり、NetScalerは VIP でのクライアント接続を終了してから、 (ローカル) ロード・バランシング・サービスで構成されたサーバーとの接続を開始します。 
* **DNS (ドメイン・ネーム・システム) およびネーム・サーバー:** GSLB での名前解決は、通常の DNS とほぼ同様に機能します。 違いは、解決されたアドレスを判別するために使用するロジック/基準です。GSLBは、事前構成されたロード・バランシング方式を使用してこの解決を処理します。 NetScaler は、さまざまな方法で DNS と対話するように構成できます。
	* 権限 DNS (ADNS)。 ADNS モードを使用する NetScalers は、特定のドメインおよびそのドメイン上のすべてのレコードに対して権限を持ちます。
	* DNS サブ委任。 DNS サーバー (ドメインの権限) がサブドメインの責任を NetScaler システムに委任するときに発生します。
	* DNS プロキシー。 このモードで構成されている場合、NetScaler は名前解決を処理する別のサーバー (外部) に DNS 要求を転送します。
* **メソッド:** メソッドは、GSLB 仮想サーバーがトポロジーから最適な GSLB サービスを選択するために使用するアルゴリズムです。 アルゴリズムは、実際の選択基準に対応するパフォーマンスの側面を評価します。 以下のメソッドを使用できます。
  * ラウンドロビン: 負荷に関係なく、GSLB サイト/サービス間で、処理する着信要求を順に割り振ります。
  * 往復時間 (RTT): クライアントのローカル DNS サーバーと (GSLB) サイトとの間の時間または遅延の指標。 NetScaler は、さまざまなメカニズム (ICMP エコー要求/応答 (PING)、UDP、TCP など) を使用して RTT メトリックを収集します。
  * 静的近接性: IP アドレス・データベースを使用して、クライアントのローカル DNS サーバーとサイトとの間の接近性を確立し、それを選択用の基準として使用します。
  * 最小接続数: 選択基準として、各サイト/サービスのアクティブ接続の最小数を測定します。
  * 最小応答時間: アクティブ接続が最も少なくて平均応答時間が最も短いサイトを選択します。
  * 最小帯域幅: 現在、最も少ないトラフィック (Mbps) を管理しているサイトを選択します。
  * 最小パケット数: 過去 14 秒間に受信したパケット数が最も少ないサービスを選択します。
  * ソース IP アドレス・ハッシュ: クライアントの IPv4 アドレスまたは IPv6 アドレスのハッシュ値に基づいてサービスを選択します。
  * カスタム・ロード: アクティブなトランザクションをまったく処理していないサービスを選択します。 ロード・バランシング・セットアップ内のすべてのサービスがアクティブ・トランザクションを処理している場合、アプライアンスは負荷 (CPU 使用量、メモリー、応答時間) が最も小さいサービスを選択します。

* **MEP (Metric Exchange Protocol):** サイト間でメトリック (負荷およびネットワーク) とパーシスタンス情報を交換するために使用される専有プロトコル。 MEP は、GSLB メッシュ/トポロジー内のさまざまなサイト/NetScaler 間のヘルス・チェックを提供します。 MEP は、管理者によって設定された基準を使用して、以前に構成された選択パラメーターに基づいてサイトが通信してトラフィックを処理する方法を提供します。 MEP は TCP ポート 3009 と 3011 を使用します。 MEP が使用不可の場合、メソッドの選択は、アスタリスク (*) でマークされたものよりも前にリストされたオプションに限定されます。 他のメソッドを選択すると、ラウンドロビンに戻されます。
* **モニタリング:** NetScaler エンジンは、MEP、または問題のサービスにバインドされた明示的なモニターを使用して、リモート GSLB サービスの状態を定期的に評価します。 モニターは、通常のロード・バランシング・サービスと同様に使用されます。 GSLB の場合、通常は MEP によって制御されるため、ローカル・サービスにモニターを追加する必要はありません。 
* **パーシスタンス:** 特定のドメインのサイト・プリファレンスを確立するオプション・フィーチャーです。 この特定のユース・ケースでは、トラフィックはロード・バランシングされず、同じデータ・センターによって処理されます。 これは、トランザクション・データがサイト/サーバーごとに固有である、e-コマースなどの特定のアプリケーションに役立ちます。
* **GSLB サイト:** サイトは、NetScaler システムが構成されている/存在するデータ・センター (ロケーション) として定義できます。 各 GSLB サイトは、そのサイトにとって「ローカル」と見なされる NetScaler システムによって管理される一方、他のすべてのリモート・システム/サイトは「リモート」サイトと見なされて処理されます。
* **GSLB サービス:** (通常の) 仮想サーバー/VIP を表し、(かつ仮想サーバー/VIPにバインドされる) オブジェクトです。 ローカル (同一サイト) またはリモートのいずれかです。
* **GSLB 仮想サーバー:** GSLB 仮想サーバーは、1 つ以上の GSLB サービスに割り当てられます。通常、この GSLB サービスは別の (GSLB) サイトの一部です。 そこで受信したトラフィックは、バインドされたサイト間でロード・バランシングされます。 サイトの選択は、メソッドとユース・ケースに基づいて行われます。
* **GSLB ドメイン:** GSLB 仮想サーバーが担当するドメインまたはゾーンを表します。 
* **ADNS サービス:** IP アドレスとポートの組み合わせによって表されるサービスです。このサービスで、NetScaler が権限を持つドメインへの DNS 要求が送信されます。 NetScaler は応答を取得するために、そのドメインにバインドされた GSLB 仮想サーバーに DNS 要求を取り込みます。
