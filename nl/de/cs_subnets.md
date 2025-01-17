---

copyright:
  years: 2014, 2019
lastupdated: "2019-06-05"

keywords: kubernetes, iks

subcollection: containers

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:download: .download}
{:preview: .preview}



# Teilnetze für Cluster konfigurieren
{: #subnets}

Ändern Sie den Pool der verfügbaren, portierbaren öffentlichen oder privaten IP-Adressen für Netzausgleichsfunktions- (NLB-) Services, indem Sie Ihrem Kubernetes-Cluster Teilnetze hinzufügen.
{:shortdesc}



## Übersicht über den Netzbetrieb in {{site.data.keyword.containerlong_notm}}
{: #basics}

Machen Sie sich mit den Basiskonzepten des Netzbetriebs in {{site.data.keyword.containerlong_notm}}-Clustern vertraut. {{site.data.keyword.containerlong_notm}} verwendet VLANs, Teilnetze und IP-Adressen, um Clusterkomponenten mit Netzkonnektivität auszustatten.
bernetes-Cluster Teilnetze hinzufügen.
{: shortdesc}

### VLANs
{: #basics_vlans}

Beim Erstellen eines Clusters werden die Workerknoten des Clusters automatisch mit einem VLAN verbunden. Ein VLAN konfiguriert eine Gruppe von Workerknoten und Pods so, als wären diese an dasselbe physische Kabel angeschlossen und es bietet einen Kanal für die Konnektivität zwischen Worker und Pods.
{: shortdesc}

<dl>
<dt>VLANs für kostenlose Cluster</dt>
<dd>Bei kostenlosen Clustern werden die Workerknoten des Clusters standardmäßig mit einem öffentlichen und einem privaten VLAN verbunden, deren Eigner IBM ist. Da IBM die VLANs, Teilnetze und IP-Adressen steuert, können Sie keine Mehrzonencluster erstellen oder Ihrem Cluster Teilnetze hinzufügen; darüber hinaus können Sie zum Verfügbarmachen Ihrer App nur NodePort-Services verwenden.</dd>
<dt>VLANs für Standardcluster</dt>
<dd>Wenn Sie bei Standardclustern in einer Zone zum ersten Mal einen Cluster erstellen, werden in dieser Zone automatisch ein öffentliches und ein privates VLAN für Sie in Ihrem Konto der IBM Cloud-Infrastruktur (SoftLayer) bereitgestellt. Für jeden weiteren Cluster, den Sie in dieser Zone erstellen, müssen Sie das VLAN-Paar angeben, das Sie in dieser Zone verwenden möchten. Sie können dasselbe öffentliche und private VLAN wiederverwenden, die für Sie erstellt wurden, da ein VLAN von mehreren Clustern gemeinsam genutzt werden kann.</br>
</br>Sie können Ihre Workerknoten entweder mit einem öffentlichen und dem privaten VLAN verbinden oder nur mit dem privaten VLAN. Wenn Ihre Workerknoten nur mit einem privaten VLAN verbunden werden sollen, können Sie die ID eines vorhandenen privaten VLANs verwenden oder [ein privates VLAN erstellen](/docs/cli/reference/ibmcloud?topic=cloud-cli-manage-classic-vlans#sl_vlan_create) und die ID während der Clustererstellung verwenden.</dd></dl>

Um die VLANs anzuzeigen, die in den einzelnen Zonen für Ihr Konto bereitgestellt sind, führen Sie den Befehl `ibmcloud ks vlans --zone <zone>` aus. Um die VLANs anzuzeigen, in denen ein Cluster bereitgestellt ist, führen Sie den Befehl `ibmcloud ks cluster-get --cluster <clustername_oder_-id> --showResources` aus und suchen nach dem Abschnitt für VLANs von Teilnetzen (**Subnet VLANs**).

Die IBM Cloud-Infrastruktur (SoftLayer) verwaltet die VLANs, die automatisch bereitgestellt werden, wenn Sie Ihren ersten Cluster in einer Zone erstellen. Wenn ein VLAN nicht mehr verwendet wird, z. B. nach dem Entfernen aller Workerknoten aus dem VLAN, wird das VLAN von der IBM Cloud-Infrastruktur (SoftLayer) freigegeben. Wenn Sie ein neues VLAN benötigen, [wenden Sie sich an den {{site.data.keyword.Bluemix_notm}}-Support](/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#ordering-premium-vlans).

**Kann ich meine VLAN-Entscheidung später ändern?**</br>
Sie können Ihre VLAN-Konfiguration ändern, indem Sie die Worker-Pools in Ihrem Cluster modifizieren. Weitere Informationen finden Sie unter [VLAN-Verbindungen für Workerknoten ändern](/docs/containers?topic=containers-cs_network_cluster#change-vlans).

### Teilnetze un IP-Adressen
{: #basics_subnets}

Neben Workerknoten und Pods werden auch Teilnetze automatisch in VLANs bereitgestellt. Teilnetze stellen für Ihre Clusterkomponenten Netzkonnektivität bereit, indem sie ihnen IP-Adressen zuordnen.
{: shortdesc}

Folgende Teilnetze werden in den standardmäßigen öffentlichen und privaten VLANs automatisch bereitgestellt:

**Öffentliche VLAN-Teilnetze**
* Durch das primäre öffentliche Teilnetz werden die öffentlichen IP-Adressen festgelegt, die während der Clustererstellung den Workerknoten zugeordnet werden. Mehrere im selben VLAN vorhandene Cluster können ein einzelnes primäres öffentliches Teilnetz gemeinsam nutzen.
* Das portierbare öffentliche Teilnetz ist nur an einen einzigen Cluster gebunden und stellt dem Cluster acht öffentliche IP-Adressen zur Verfügung. 3 IP-Adressen sind für Funktionen der IBM Cloud-Infrastruktur (SoftLayer) reserviert. 1 IP wird von der standardmäßigen öffentlichen Ingress-ALB verwendet und 4 IPs können zum Erstellen von öffentlichen NLB-Services (NLB = Network Load Balancer) oder weiteren öffentlichen ALBs verwendet werden. Portierbare öffentliche IPs sind permanente, feste IP-Adressen, die für den Zugriff auf NLBs oder ALBs über das Internet verwendet werden können. Wenn Sie für NLBs oder ALBs mehr als 4 IPs benötigen, lesen Sie die Informationen unter [Portierbare IP-Adressen hinzufügen](/docs/containers?topic=containers-subnets#adding_ips).

**Private VLAN-Teilnetze**
* Durch das primäre private Teilnetz werden die privaten IP-Adressen festgelegt, die während der Clustererstellung den Workerknoten zugeordnet werden. Mehrere im selben VLAN vorhandene Cluster können ein einzelnes primäres privates Teilnetz gemeinsam nutzen.
* Das portierbare private Teilnetz ist nur an einen einzigen Cluster gebunden und stellt dem Cluster acht private IP-Adressen zur Verfügung. 3 IP-Adressen sind für Funktionen der IBM Cloud-Infrastruktur (SoftLayer) reserviert. 1 IP wird von der standardmäßigen privaten Ingress-ALB verwendet und 4 IPs können zum Erstellen von privaten NLB-Services (NLB = Network Load Balancer) oder weiteren privaten ALBs verwendet werden. Portierbare private IPs sind permanente, feste IP-Adressen, die für den Zugriff auf NLBs oder ALBs über ein privates Netz verwendet werden können. Wenn Sie für private NLBs oder ALBs mehr als 4 IPs benötigen, lesen Sie die Informationen unter [Portierbare IP-Adressen hinzufügen](/docs/containers?topic=containers-subnets#adding_ips).

Führen Sie `ibmcloud ks subnets` aus, um alle in Ihrem Konto bereitgestellten Teilnetze anzuzeigen. Um die portierbaren öffentlichen und die portierbaren privaten Teilnetze anzuzeigen, die an einen einzigen Cluster gebunden sind, können Sie den Befehl `ibmcloud ks cluster-get --cluster <clustername_oder_-id> --showResources` ausführen und nach dem Abschnitt für VLANs von Teilnetzen (**Subnet VLANs**) suchen.

In {{site.data.keyword.containerlong_notm}} haben VLANs ein Limit von 40 Teilnetzen. Überprüfen Sie bei Erreichen dieses Grenzwerts zunächst, ob Sie [Teilnetze im VLAN wiederverwenden können, um neue Cluster zu erstellen](/docs/containers?topic=containers-subnets#subnets_custom). Wenn Sie ein neues VLAN benötigen, fordern Sie eines an, indem Sie den [{{site.data.keyword.Bluemix_notm}}-Support kontaktieren](/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#ordering-premium-vlans). [Erstellen Sie dann einen Cluster](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_cluster_create), der dieses neue VLAN verwendet.
{: note}

**Ändert sich die IP-Adresse für meine Workerknoten?**</br>
Ihrem Workerknoten wird eine IP-Adresse für die öffentlichen oder privaten VLANs zugewiesen, die vom Cluster verwendet werden. Nach der Bereitstellung des Workerknotens ändern sich die IP-Adressen nicht mehr. Die IP-Adressen des Workerknotens bleiben beispielsweise bei den Operationen `reload`, `reboot` und `update` bestehen. Darüber hinaus wird die private IP-Adresse des Workerknotens in den meisten `kubectl`-Befehlen für die Identität des Workerknotens verwendet. Wenn Sie die vom Worker-Pool verwendeten VLANs ändern, verwenden neue Workerknoten, die in diesem Pool bereitgestellt werden, die neuen VLANs für ihre IP-Adressen. Bestehende IP-Adressen von Workerknoten ändern sich nicht, Sie können jedoch entscheiden, die Workerknoten, die die alten VLANs verwenden, zu entfernen.

### Netzsegmentierung
{: #basics_segmentation}

Mit 'Netzsegmentierung' wird die Methode beschrieben, bei der ein Netz in mehrere Teilnetze aufgeteilt wird. Apps, die in einem Teilnetz ausgeführt werden, können Apps in einem anderen Teilnetz nicht anzeigen oder auf sie zugreifen. Weitere Informationen zu Optionen der Netzsegmentierung und ihrer Beziehung zu VLANs finden Sie [in diesem Abschnitt zur Clustersicherheit](/docs/containers?topic=containers-security#network_segmentation).
{: shortdesc}

In verschiedenen Situationen müssen Komponenten in Ihrem Cluster jedoch berechtigt werden, über mehrere private VLANs hinzu zu kommunizieren. Wenn Sie zum Beispiel einen Mehrzonencluster erstellen wollen, wenn Sie mehrere VLANs für einen Cluster haben oder wenn Sie mehrere Teilnetze im selben VLAN haben, können die Workerknoten in verschiedenen Teilnetzen desselben VLAN oder in verschiedenen VLANs nicht automatisch miteinander kommunizieren. Sie müssen entweder eine VRF-Funktion (Virtual Router Function) oder das VLAN-Spanning für Ihr IBM Cloud-Infrastrukturkonto (SoftLayer) aktivieren.

**Was sind VRF-Funktionen und VLAN-Spanning?**</br>

<dl>
<dt>[Virtual Router Function (VRF)](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud)</dt>
<dd>Eine VRF-Funktion ermöglicht allen VLANs und Teilnetzen in Ihrem Infrastrukturkonto die Kommunikation miteinander. Darüber hinaus ist eine VRF-Funktion erforderlich, um die Kommunikation Ihrer Worker und Ihres Masters über den privaten Serviceendpunkt zu ermöglichen. Zur Aktivierung von VRF [wenden Sie sich an Ihren Ansprechpartner für die IBM Cloud-Infrastruktur (SoftLayer)](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#how-you-can-initiate-the-conversion). Beachten Sie, dass VRF die Option des VLAN-Spannings für Ihr Konto ausschließt, da alle VLANs kommunizieren können, sofern Sie keine Gateway-Einheit zur Verwaltung des Datenverkehrs konfigurieren.</dd>
<dt>[VLAN-Spanning](/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)</dt>
<dd>Wenn Sie VRF nicht aktivieren können oder wollen, aktivieren Sie das VLAN-Spanning. Um diese Aktion durchführen zu können, müssen Sie über die [Infrastrukturberechtigung](/docs/containers?topic=containers-users#infra_access) **Netz > VLAN-Spanning im Netz verwalten** verfügen oder Sie können den Kontoeigner bitten, diese zu aktivieren. Zum Prüfen, ob das VLAN-Spanning bereits aktiviert ist, verwenden Sie den [Befehl](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get --region <region>`. Beachten Sie, dass Sie den privaten Serviceendpunkt nicht aktivieren können, wenn Sie das VLAN-Spanning anstatt einer VRF-Funktion aktivieren.</dd>
</dl>

**Wie wirkt sich eine VRF-Funktion oder das VLAN-Spanning auf die Netzsegmentierung aus?**</br>

Wenn eine VRF-Funktion oder das VLAN-Spanning aktiviert ist, kann jedes System, das mit einem der privaten VLANs im selben {{site.data.keyword.Bluemix_notm}}-Konto verbunden ist, mit Workerknoten kommunizieren. Sie können Ihren Cluster von anderen Systemen im privaten Netz mithilfe von [Calico-Richtlinien für private Netze](/docs/containers?topic=containers-network_policies#isolate_workers) isolieren. {{site.data.keyword.containerlong_notm}} ist darüber hinaus mit allen [Firewallangeboten der IBM Cloud-Infrastruktur (SoftLayer) ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://www.ibm.com/cloud-computing/bluemix/network-security) kompatibel. Sie können eine Firewall wie [Virtual Router Appliance](/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra) mit angepassten Netzrichtlinien einrichten, um für Ihren Standardcluster dedizierte Netzsicherheit bereitzustellen und unbefugten Zugriff zu erkennen und zu unterbinden.

<br />



## Vorhandene Teilnetze zum Erstellen eines Clusters verwenden
{: #subnets_custom}

Wenn Sie einen Standardcluster erstellen, werden Teilnetze automatisch für Sie erstellt. Statt die automatisch bereitgestellten Teilnetze zu verwenden, können Sie jedoch vorhandene portierbare Teilnetze aus Ihrem Konto der IBM Cloud-Infrastruktur (SoftLayer) verwenden oder Teilnetze aus einem gelöschten Cluster wiederverwenden.
{:shortdesc}

Verwenden Sie diese Option, um stabile statische IP-Adressen beizubehalten, obwohl Cluster entfernt oder erstellt werden, oder um größere Blöcke von IP-Adressen zu bestellen. Wenn Sie stattdessen mehr portierbare private IP-Adressen für die Netzausgleichsfunktions- (NLB-) Services für Ihren Cluster durch Verwenden Ihres eigenen lokalen Teilnetzes erhalten wollen, lesen Sie die Informationen unter [Portierbare private IP-Adressen durch Hinzufügen benutzerverwalteter Teilnetze zu privaten VLANs hinzufügen](#subnet_user_managed).

Portierbare öffentliche IP-Adressen werden monatlich berechnet. Wenn Sie nach der Bereitstellung Ihres Clusters portierbare öffentliche IP-Adressen entfernen, müssen Sie trotzdem die monatliche Gebühr bezahlen, auch wenn sie sie nur über einen kurzen Zeitraum genutzt haben.
{: note}

Vorbereitende Schritte:
- [Melden Sie sich an Ihrem Konto an. Geben Sie, sofern anwendbar, die richtige Ressourcengruppe als Ziel an. Legen Sie den Kontext für den Cluster fest.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)
- Wenn Sie Teilnetze aus einem Cluster wiederverwenden möchten, den Sie nicht mehr benötigen, löschen Sie den nicht benötigten Cluster. Erstellen Sie den neuen Cluster sofort, da die Teilnetze innerhalb von 24 Stunden gelöscht werden, wenn Sie sie nicht wiederverwenden.

   ```
   ibmcloud ks cluster-rm --cluster <clustername_oder_-id>
   ```
   {: pre}

</br>Gehen Sie wie folgt vor, um ein vorhandenes Teilnetz in Ihrem Portfolio der IBM Cloud-Infrastruktur (SoftLayer) zu verwenden:

1. Rufen Sie die ID des zu verwendenden Teilnetzes ab sowie die ID des VLANs, in dem sich das Teilnetz befindet.

    ```
    ibmcloud ks subnets
    ```
    {: pre}

    In dieser Beispielausgabe lautet die Teilnetz-ID `1602829` und die VLAN-ID `2234945`:
    ```
    Getting subnet list...
    OK
    ID        Network             Gateway          VLAN ID   Type      Bound Cluster
    1550165   10.xxx.xx.xxx/26    10.xxx.xx.xxx    2234947   private
    1602829   169.xx.xxx.xxx/28   169.xx.xxx.xxx   2234945   public
    ```
    {: screen}

2. Erstellen Sie unter Verwendung der von Ihnen angegebenen VLAN-ID [einen Cluster in der CLI](/docs/containers?topic=containers-clusters#clusters_cli_steps). Schließen Sie das Flag `--no-subnet` ein, um zu verhindern, dass ein neues portierbares öffentliches IP-Teilnetz und ein neues portierbares privates IP-Teilnetz automatisch erstellt werden.

    ```
    ibmcloud ks cluster-create --zone dal10 --machine-type b3c.4x16 --no-subnet --public-vlan 2234945 --private-vlan 2234947 --workers 3 --name my_cluster
    ```
    {: pre}
    Wenn Sie sich nicht erinnern können, in welcher Zone sich das VLAN für das Flag `--zone` befindet, können Sie überprüfen, ob sich das VLAN in einer bestimmten Zone befindet, indem Sie den Befehl `ibmcloud ks vlans --zone <zone>` ausführen.
    {: tip}

3.  Überprüfen Sie, dass der Cluster erstellt wurde. Es kann bis zu 15 Minuten dauern, bis die Maschinen mit den Workerknoten angewiesen werden und der Cluster in Ihrem Konto eingerichtet und bereitgestellt wird.

    ```
    ibmcloud ks clusters
    ```
    {: pre}

    Wenn Ihr Cluster vollständig bereitgestellt ist, ändert sich der **Status** in `deployed` (bereitgestellt).

    ```
    Name         ID                                   State      Created          Workers    Zone      Version     Resource Group Name
    mycluster    aaf97a8843a29941b49a598f516da72101   deployed   20170201162433   3          dal10     1.13.6      Default
    ```
    {: screen}

4.  Überprüfen Sie den Status der Workerknoten.

    ```
    ibmcloud ks workers --cluster <clustername_oder_-id>
    ```
    {: pre}

    Bevor Sie mit dem nächsten Schritt fortfahren, müssen die Workerknoten bereit sein. Der **Status** ändert sich in `normal` und der **Status** lautet `Ready` (Bereit).

    ```
    ID                                                  Public IP        Private IP     Machine Type   State      Status   Zone     Version
    prod-dal10-pa8dfcc5223804439c87489886dbbc9c07-w1    169.xx.xxx.xxx   10.xxx.xx.xxx  free           normal     Ready    dal10      1.13.6
    ```
    {: screen}

5.  Fügen Sie Ihrem Cluster das Teilnetz hinzu, indem Sie die Teilnetz-ID angeben. Wenn Sie ein Teilnetz für einen Cluster verfügbar machen, wird eine Kubernetes-Konfigurationszuordnung für Sie erstellt, die alle verfügbaren portierbaren öffentlichen IP-Adressen enthält, die Sie verwenden können. Wenn in der Zone, in der sich das VLAN des Teilnetzes befindet, keine Ingress-ALBs (Lastausgleichsfunktion für Anwendungen) vorhanden sind, wird automatisch eine portierbare öffentliche und eine portierbare private IP-Adresse verwendet, um die öffentlichen und privaten ALBs für diese Zone zu erstellen. Sie können alle anderen portierbaren öffentlichen und privaten IP-Adressen aus dem Teilnetz verwenden, um für Ihre Apps NLB-Services zu erstellen.

  ```
  ibmcloud ks cluster-subnet-add --cluster <clustername_oder_-id> --subnet-id <teilnetz-id>
  ```
  {: pre}

  Beispielbefehl:
  ```
  ibmcloud ks cluster-subnet-add --cluster mycluster --subnet-id 807861
  ```
  {: screen}

6. **Wichtig**: Um die Kommunikation zwischen Workern zu aktivieren, die sich in unterschiedlichen Teilnetzen im selben VLAN befinden, müssen Sie [das Routing zwischen Teilnetzen im selben VLAN aktivieren](#subnet-routing).

<br />


## Vorhandene portierbare IP-Adressen verwalten
{: #managing_ips}

Die 4 portierbaren öffentlichen und 4 portierbaren privaten IP-Adressen können verwendet werden, um einzelne Apps für das öffentliche oder private Netz verfügbar zu machen, indem Sie einen [Netzausgleichsfunktions-Service (NLB-Service) erstellen](/docs/containers?topic=containers-loadbalancer) oder [weitere Ingress-ALBs (Application Load Balancer) erstellen](/docs/containers?topic=containers-ingress#scale_albs). Um einen NLB- oder ALB-Service zu erstellen, muss mindestens 1 portierbare IP-Adresse des richtigen Typs für Sie verfügbar sein. Sie können portierbare IP-Adressen anzeigen, die verfügbar sind, oder eine bereits verwendete portierbare IP-Adresse freigeben.
{: shortdesc}

### Verfügbare portierbare öffentliche IP-Adressen anzeigen
{: #review_ip}

Um sowohl verwendete als auch verfügbare portierbare IP-Adressen in Ihrem Cluster aufzulisten, können Sie den folgenden Befehl ausführen.
{: shortdesc}

```
kubectl get cm ibm-cloud-provider-vlan-ip-config -n kube-system -o yaml
```
{: pre}

Wenn Sie nur portierbare öffentliche IP-Adressen auflisten möchten, die zum Erstellen von öffentlichen NLBs oder weiteren öffentlichen ALBs zur Verfügung stehen, können Sie die folgenden Schritte ausführen:

Vorbereitende Schritte:
-  Stellen Sie sicher, dass Sie die [{{site.data.keyword.Bluemix_notm}} IAM-Servicerolle **Schreibberechtigter** oder **Manager**](/docs/containers?topic=containers-users#platform) für den Namensbereich `default` innehaben.
- [Melden Sie sich an Ihrem Konto an. Geben Sie, sofern anwendbar, die richtige Ressourcengruppe als Ziel an. Legen Sie den Kontext für den Cluster fest.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)

Gehen Sie wie folgt vor, um ver verfügbaren portierbaren öffentlichen IP-Adressen aufzulisten:

1.  Erstellen Sie eine Kubernetes Service-Konfigurationsdatei namens `myservice.yaml` und definieren Sie einen Service vom Typ `LoadBalancer` mit einer Dummy-IP-Adresse für die NLB. Im folgenden Beispiel wird die IP-Adresse '1.1.1.1' als NLB-IP-Adresse verwendet. Ersetzen Sie `<zone>` durch die Zone, in der die verfügbaren IP-Adressen geprüft werden sollen.

    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: myservice
      name: myservice
      namespace: default
      annotations:
        service.kubernetes.io/ibm-load-balancer-cloud-provider-zone: "<zone>"
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: myservice
      sessionAffinity: None
      type: LoadBalancer
      loadBalancerIP: 1.1.1.1
    ```
    {: codeblock}

2.  Erstellen Sie den Service in Ihrem Cluster.

    ```
    kubectl apply -f myservice.yaml
    ```
    {: pre}

3.  Überprüfen Sie den Service.

    ```
    kubectl describe service myservice
    ```
    {: pre}

    Die Erstellung dieses Service schlägt fehl, weil der Kubernetes-Master die angegebene NLB-IP-Adresse in der Kubernetes-Konfigurationszuordnung nicht finden kann. Wenn Sie diesen Befehl ausführen, können Sie die Fehlernachricht und eine Liste der verfügbaren öffentlichen IP-Adressen für den Cluster anzeigen.

    ```
    Error on cloud load balancer a8bfa26552e8511e7bee4324285f6a4a for service default/myservice with UID 8bfa2655-2e85-11e7-bee4-324285f6a4af: Requested cloud provider IP 1.1.1.1 is not available. The following cloud provider IP addresses are available: <liste_von_IP-adressen>
    ```
    {: screen}

<br />


### Verwendete IP-Adressen freigeben
{: #free}

Sie können eine bereits verwendete portierbare IP-Adresse freigeben, indem Sie den Netzausgleichsfunktionsservice (NLB-Service) löschen oder die Ingress-Netzlastausgleichsfunktion (ALB) inaktivieren, der/die die portierbare IP-Adresse belegt.
{:shortdesc}

Vorbereitende Schritte:
-  Stellen Sie sicher, dass Sie die [{{site.data.keyword.Bluemix_notm}} IAM-Servicerolle **Schreibberechtigter** oder **Manager**](/docs/containers?topic=containers-users#platform) für den Namensbereich `default` innehaben.
- [Melden Sie sich an Ihrem Konto an. Geben Sie, sofern anwendbar, die richtige Ressourcengruppe als Ziel an. Legen Sie den Kontext für den Cluster fest.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)

Gehen Sie wie folgt vor, um eine NLB zu löschen oder eine ALB zu inaktivieren:

1. Listen Sie verfügbare Services in Ihrem Cluster auf.
    ```
    kubectl get services | grep LoadBalancer
    ```
    {: pre}

2. Entfernen Sie den Lastausgleichsservice oder inaktivieren Sie die ALB, der/die eine öffentliche oder private IP-Adresse belegt.
  * Löschen Sie eine NLB:
    ```
    kubectl delete service <servicename>
    ```
    {: pre}
  * Inaktivieren Sie eine ALB:
    ```
    ibmcloud ks alb-configure --albID <ALB-ID> --disable
    ```
    {: pre}

<br />


## Portierbare IP-Adressen hinzufügen
{: #adding_ips}

Die 4 portierbaren öffentlichen und 4 portierbaren privaten IP-Adressen können verwendet werden, um einzelne Apps für das öffentliche oder private Netz verfügbar zu machen, indem Sie einen [Netzausgleichsfunktions-Service (NLB-Service) erstellen](/docs/containers?topic=containers-loadbalancer). Wenn Sie mehr als vier öffentliche oder vier private NLBs erstellen möchten, können Sie weitere portierbare IP-Adressen erhalten, indem Sie dem Cluster Netz-Teilnetze hinzufügen.
{: shortdesc}

Wenn Sie ein Teilnetz in einem Cluster verfügbar machen, werden IP-Adressen dieses Teilnetzes zum Zweck von Clusternetzen verwendet. Vermeiden Sie IP-Adresskonflikte, indem Sie ein Teilnetz mit nur einem Cluster verwenden. Verwenden Sie kein Teilnetz für mehrere Cluster oder für andere
Zwecke außerhalb von {{site.data.keyword.containerlong_notm}} gleichzeitig.
{: important}

### Portierbare IPs hinzufügen, indem Sie weitere Teilnetze bestellen
{: #request}

Sie können für NLB-Services weitere portierbare IPs erhalten, indem Sie in einem Konto der IBM Cloud-Infrastruktur (SoftLayer) ein neues Teilnetz erstellen und es für den angegebenen Cluster verfügbar machen.
{:shortdesc}

Portierbare öffentliche IP-Adressen werden monatlich berechnet. Wenn Sie nach der Bereitstellung Ihres Teilnetzes portierbare öffentliche IP-Adressen entfernen, müssen Sie trotzdem die monatliche Gebühr bezahlen, auch wenn sie sie nur für einen kurzen Zeitraum genutzt haben.
{: note}

Vorbereitende Schritte:
-  Stellen Sie sicher, dass Sie die [{{site.data.keyword.Bluemix_notm}} IAM-Plattformrolle **Operator** oder **Administrator**](/docs/containers?topic=containers-users#platform) für den Cluster innehaben.
- [Melden Sie sich an Ihrem Konto an. Geben Sie, sofern anwendbar, die richtige Ressourcengruppe als Ziel an. Legen Sie den Kontext für den Cluster fest.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)

Gehen Sie wie folgt vor, um ein Teilnetz zu bestellen:

1. Stellen Sie ein neues Teilnetz bereit.

    ```
    ibmcloud ks cluster-subnet-create --cluster <clustername_oder_-id> --size <teilnetzgröße> --vlan <vlan-id>
    ```
    {: pre}

    <table>
    <caption>Erklärung der Bestandteile dieses Befehls</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Ideensymbol"/> Erklärung der Bestandteile dieses Befehls</th>
    </thead>
    <tbody>
    <tr>
    <td><code><em>&lt;clustername_oder_-id&gt;</em></code></td>
    <td>Ersetzen Sie <code>&lt;clustername_oder_-id&gt;</code> durch den Namen oder die ID des Clusters.</td>
    </tr>
    <tr>
    <td><code><em>&lt;teilnetzgröße&gt;</em></code></td>
    <td>Ersetzen Sie <code>&lt;teilnetzgröße&gt;</code> durch die Anzahl der IP-Adressen, die Sie im portierbaren Teilnetz hinzufügen wollen. Gültige Werte sind 8, 16, 32 oder 64. <p class="note"> Wenn Sie portierbare IP-Adressen für Ihr Teilnetz hinzufügen, werden drei IP-Adressen verwendet, um clusterinterne Netze einzurichten. Diese drei IP-Adressen können nicht für Ihre Ingress-Lastausgleichsfunktionen für Anwendungen (ALBs) oder zum Erstellen von Netzausgleichsfunktions- (NLB-) Services verwendet werden. Wenn Sie beispielsweise acht portierbare öffentliche IP-Adressen anfordern, können Sie fünf verwenden, um die Apps öffentlich verfügbar zu machen.</p> </td>
    </tr>
    <tr>
    <td><code><em>&lt;VLAN_ID&gt;</em></code></td>
    <td>Ersetzen Sie <code>&lt;VLAN_ID&gt;</code> durch die ID des öffentlichen oder privaten VLANs, in dem Sie die portierbaren öffentlichen oder privaten IP-Adressen zuweisen möchten. Sie müssen ein öffentliches oder privates VLAN auswählen, mit dem ein vorhandener Workerknoten verbunden ist. Um die öffentlichen oder privaten VLANs zu prüfen, mit denen Ihre Workerknoten verbunden sind, führen Sie den Befehl <code>ibmcloud ks cluster-get --cluster &lt;cluster&gt; --showResources</code> aus suchen in der Ausgabe nach dem Abschnitt für VLANs von Teilnetzen (<strong>Subnet VLANs</strong>). Das Teilnetz wird in derselben Zone bereitgestellt, in der sich das VLAN befindet.</td>
    </tr>
    </tbody></table>

2. Überprüfen Sie, ob das Teilnetz erfolgreich erstellt und zu Ihrem Cluster hinzugefügt wurde. Das Teilnetz-CIDR wird im Abschnitt für **Teilnetz-VLANs** aufgeführt.

    ```
    ibmcloud ks cluster-get --showResources <clustername_oder_-id>
    ```
    {: pre}

    In dieser Beispielausgabe wurde dem öffentlichen VLAN `2234945` ein zweites Teilnetz hinzugefügt:
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR          Public   User-managed
    2234947   10.xxx.xx.xxx/29     false    false
    2234945   169.xx.xxx.xxx/29    true     false
    2234945   169.xx.xxx.xxx/29    true     false
    ```
    {: screen}

3. **Wichtig**: Um die Kommunikation zwischen Workern zu aktivieren, die sich in unterschiedlichen Teilnetzen im selben VLAN befinden, müssen Sie [das Routing zwischen Teilnetzen im selben VLAN aktivieren](#subnet-routing).

<br />


### Portierbare private IP-Adressen durch Hinzufügen benutzerverwalteter Teilnetze zu privaten VLANs hinzufügen
{: #subnet_user_managed}

Sie können für Netzausgleichsfunktions- (NLB-) Services weitere portierbare private IPs erhalten, indem Sie ein Teilnetz von einem lokalen Netz für Ihren Cluster verfügbar machen.
{:shortdesc}

Wollen Sie stattdessen vorhandene portierbare Teilnetze in Ihrem Konto für die IBM Cloud-Infrastruktur (SoftLayer) wiederverwenden? Weitere Informationen finden Sie unter [Angepasste oder vorhandene Teilnetze der IBM Cloud-Infrastruktur (SoftLayer) für die Erstellung eines Clusters verwenden](#subnets_custom).
{: tip}

Voraussetzungen:
- Vom Benutzer verwaltete Teilnetze können nur zu privaten VLANs hinzugefügt werden.
- Die Begrenzung für die Länge des Teilnetzpräfix beträgt /24 bis /30. Beispiel: `169.xx.xxx.xxx/24` gibt 253 verwendbare private IP-Adressen an, während `169.xx.xxx.xxx/30` eine verwendbare private IP-Adresse angibt.
- Die erste IP-Adresse im Teilnetz muss als Gateway für das Teilnetz verwendet werden.

Vorbereitende Schritte:
- Konfigurieren Sie vorab das Routing des Netzverkehrs in das externe Teilnetz bzw. aus dem externen Teilnetz.
- Vergewissern Sie sich, dass Sie über VPN-Konnektivität zwischen dem Netzgateway des lokalen Rechenzentrums und entweder der Virtual Router Appliance des privaten Netzes oder dem aktiven strongSwan-VPN-Service in Ihrem Cluster verfügen. Weitere Informationen finden Sie unter [VPN-Konnektivität einrichten](/docs/containers?topic=containers-vpn).
-  Stellen Sie sicher, dass Sie die [{{site.data.keyword.Bluemix_notm}} IAM-Plattformrolle **Operator** oder **Administrator**](/docs/containers?topic=containers-users#platform) für den Cluster innehaben.
- [Melden Sie sich an Ihrem Konto an. Geben Sie, sofern anwendbar, die richtige Ressourcengruppe als Ziel an. Legen Sie den Kontext für den Cluster fest.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)


Gehen Sie wie folgt vor, um ein lokales Netz im Unternehmen hinzuzufügen:

1. Zeigen Sie die ID des privaten VLANs Ihres Clusters an. Suchen Sie den Abschnitt für **Teilnetz-VLANs**. Geben Sie im Feld für die Verwaltung durch den Benutzer die VLAN-ID mit _false_ an.

    ```
    ibmcloud ks cluster-get --showResources <clustername>
    ```
    {: pre}

    ```
    Subnet VLANs
  VLAN ID   Subnet CIDR         Public   User-managed
  2234947   10.xxx.xx.xxx/29    false    false
  2234945   169.xx.xxx.xxx/29  true    false
    ```
    {: screen}

2. Fügen Sie das externe Teilnetz zu Ihrem privaten VLAN hinzu. Die portierbaren privaten IP-Adressen werden zur Konfigurationszuordnung des Clusters hinzugefügt.

    ```
    ibmcloud ks cluster-user-subnet-add --cluster <clustername> --subnet-cidr <teilnetz-CIDR> --private-vlan <privates_VLAN>
    ```
    {: pre}

    Beispiel:

    ```
    ibmcloud ks cluster-user-subnet-add --cluster mein_cluster --subnet-cidr 10.xxx.xx.xxx/24 --private-vlan 2234947
    ```
    {: pre}

3. Überprüfen Sie, ob das vom Benutzer bereitgestellte Teilnetz hinzugefügt wurde. Das Feld für die Verwaltung durch den Benutzer ist auf _true_ gesetzt.

    ```
    ibmcloud ks cluster-get --showResources <clustername>
    ```
    {: pre}

    In dieser Beispielausgabe wurde dem privaten VLAN `2234947` ein zweites Teilnetz hinzugefügt:
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR       Public   User-managed
    2234947   10.xxx.xx.xxx/29  false    false
    2234945   169.xx.xxx.xxx/29 true     false
    2234947   10.xxx.xx.xxx/24  false    true
    ```
    {: screen}

4. [Aktivieren Sie das Routing zwischen Teilnetzen im selben VLAN](#subnet-routing).

5. Fügen Sie einen [privaten Netzausgleichsfunktions- (NLB-) Service](/docs/containers?topic=containers-loadbalancer) hinzu oder aktivieren Sie die [private Ingress-ALB](/docs/containers?topic=containers-ingress#private_ingress), um über das private Netz auf Ihre App zuzugreifen. Um eine private IP-Adresse aus dem von Ihnen hinzugefügten Teilnetz zu verwenden, müssen Sie eine IP-Adresse aus dem Teilnetz-CIDR angeben. Andernfalls wird eine IP-Adresse nach dem Zufallsprinzip aus den Teilnetzen von IBM Cloud Infrastructure (SoftLayer) oder aus den vom Benutzer bereitgestellten Teilnetzen im privaten VLAN ausgewählt.

<br />


## Teilnetzrouting aktivieren
{: #subnet-routing}

Wenn Sie über mehrere VLANs für einen Cluster, mehrere Teilnetze in demselben VLAN oder einen Cluster mit mehreren Zonen verfügen, müssen Sie eine [VRF-Funktion (Virtual Router Function)](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud) für Ihr Konto für die IBM Cloud-Infrastruktur (SoftLayer) aktivieren, damit die Workerknoten über das private Netz miteinander kommunizieren können. Zur Aktivierung von VRF [wenden Sie sich an Ihren Ansprechpartner für die IBM Cloud-Infrastruktur (SoftLayer)](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#how-you-can-initiate-the-conversion). Wenn Sie VRF nicht aktivieren können oder wollen, aktivieren Sie das [VLAN-Spanning](/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning). Um diese Aktion durchführen zu können, müssen Sie über die [Infrastrukturberechtigung](/docs/containers?topic=containers-users#infra_access) **Netz > VLAN-Spanning im Netz verwalten** verfügen oder Sie können den Kontoeigner bitten, diese zu aktivieren. Zum Prüfen, ob das VLAN-Spanning bereits aktiviert ist, verwenden Sie den [Befehl](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get --region <region>`.

Sehen Sie sich folgende Szenarios an, in denen auch das VLAN-Spanning erforderlich ist.

### Routing zwischen primären Teilnetzen im selben VLAN aktivieren
{: #vlan-spanning}

Wenn Sie einen Cluster erstellen, werden ein primäres öffentliches und ein privates Teilnetz im öffentlichen und im privaten VLAN bereitgestellt. Das primäre öffentliche Teilnetz endet auf `/28` und stellt 14 öffentliche IP-Adressen für Workerknoten bereit. Das primäre private Teilnetz endet auf `/26` und stellt private IP-Adressen für bis zu 62 Workerknoten bereit.
{:shortdesc}

Es ist möglich, dass Sie die zu Anfang verfügbare Anzahl von 14 öffentlichen und 62 privaten IP-Adressen für Workerknoten überschreiten, wenn Sie einen sehr großen Cluster oder mehrere kleinere Cluster am selben Standort im selben VLAN haben. Wenn ein öffentliches oder privates Teilnetz die Grenze für Workerknoten erreicht, wird ein weiteres primäres Teilnetz in demselben VLAN bestellt.

Um sicherzustellen, dass Worker im selben VLAN in diesen primären Teilnetzen kommunizieren können, müssen Sie das VLAN-Spanning aktivieren. Entsprechende Anweisungen finden Sie in [VLAN-Spanning aktivieren oder inaktivieren](/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning).

Zum Prüfen, ob das VLAN-Spanning bereits aktiviert ist, verwenden Sie den [Befehl](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get --region <region>`.
{: tip}

### Teilnetzrouting für Gateway-Einheiten verwalten
{: #vra-routing}

Wenn Sie einen Cluster erstellen, werden ein portierbares öffentliches und ein portierbares privates Teilnetz in den VLANs angefordert, mit denen das Cluster verbunden ist. Diese Teilnetze stellen IP-Adressen für die Lastausgleichsfunktion für Anwendungen (ALB) und Netzausgleichsfunktions- (NLB-) Services bereit.
{: shortdesc}

Wenn Sie jedoch über eine vorhandene Router-Appliance verfügen, wie z. B. eine [Virtual Router Appliance (VRA)](/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about-the-vra), werden die neu hinzugefügten portierbaren Teilnetze aus diesen VLANs, mit denen der Cluster verbunden ist, nicht im Router konfiguriert. Um NLBs oder Ingress-ALBs verwenden zu können, müssen Sie sicherstellen, dass für Netzeinheiten das Routing zwischen verschiedenen Teilnetzen im selben VLAN möglich ist, indem Sie [VLAN-Spanning einrichten](/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning).

Zum Prüfen, ob das VLAN-Spanning bereits aktiviert ist, verwenden Sie den [Befehl](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get --region <region>`.
{: tip}
