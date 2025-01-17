---

copyright:
  years: 2014, 2019
lastupdated: "2019-05-31"

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




# Zugriff auf Apps mit Knotenports (NodePorts) testen
{: #nodeport}

Sie können Ihre containerisierte App für den Internetzugriff verfügbar machen, indem Sie die öffentliche IP-Adresse eines beliebigen Workerknotens in einem Kubernetes-Cluster verwenden und einen Knotenport zugänglich machen. Verwenden Sie diese Option zum Testen in {{site.data.keyword.containerlong}} und zur Bereitstellung von öffentlichem Zugriff über einen kurzen Zeitraum.
{:shortdesc}

## Netzverkehr mithilfe von Knotenports (NodePorts) verwalten
{: #nodeport_planning}

Machen Sie auf Ihrem Workerknoten einen öffentlichen Port zugänglich und verwenden Sie die öffentliche IP-Adresse des Workerknotens, um öffentlich über das Internet auf Ihren Service im Cluster zuzugreifen.
{:shortdesc}

Wenn Sie Ihre App durch Erstellen eines Kubernetes Service vom Typ 'NodePort' zugänglich machen, wird dem Service eine Knotenportnummer im Zahlenbereich 30000-32767 und eine interne Cluster-IP-Adresse zugewiesen. Der NodePort-Service fungiert als externer Einstiegspunkt für eingehende Anforderungen an die App. Der zugewiesene Knotenport (NodePort) wird in den `kubeproxy`-Einstellungen eines jeden Workerknotens im Cluster öffentlich zugänglich gemacht. Jeder Workerknoten beginnt, am zugewiesenen Knotenport empfangsbereit für eingehende Anforderungen für den Service zu sein. Um vom Internet aus auf den Service zuzugreifen, können Sie die öffentliche IP-Adresse eines beliebigen Workerknotens, die bei der Clustererstellung zugewiesen wurde, und den NodePort im Format `<IP_address>:<nodeport>` verwenden. Wenn Sie auf den Service im privaten Netz zugreifen möchten, verwenden Sie anstelle der öffentlichen IP-Adresse die private IP-Adresse eines beliebigen Workerknotens.

Das folgende Diagramm veranschaulicht, wie die Kommunikation vom Internet an eine App geleitet wird, wenn ein NodePort-Service konfiguriert ist:

<img src="images/cs_nodeport_planning.png" width="600" alt="App in {{site.data.keyword.containerlong_notm}} durch NodePort zugänglich machen" style="width:600px; border-style: none"/>

1. Eine Anforderung wird an Ihre App gesendet, indem die öffentliche IP-Adresse Ihres Workerknotens und des Knotenports (NodePort) auf dem Workerknoten verwendet wird.

2. Die Anforderung wird automatisch an die interne Cluster-IP-Adresse und den internen Port des NodePort-Service weitergeleitet. Auf die interne Cluster-IP-Adresse kann nur aus dem Cluster selbst heraus zugegriffen werden.

3. `kube-proxy` leitet die Anforderung an den Kubernetes NodePort-Service für die App weiter.

4. Die Anforderung wird an die private IP-Adresse des Pods weitergeleitet, auf dem die App bereitgestellt wird. Wenn mehrere App-Instanzen im Cluster bereitgestellt werden, leitet der NodePort-Service die Anforderungen zwischen den App-Pods weiter.

Die öffentliche IP-Adresse des Workerknotens ist nicht permanent. Wird ein Workerknoten entfernt oder neu erstellt, so wird ihm eine neue öffentliche IP-Adresse zugewiesen. Sie können den NodePort-Service verwenden, wenn Sie den öffentlichen Zugriff auf Ihre App testen möchten oder der öffentliche Zugriff nur über einen beschränkten Zeitraum erforderlich ist. Wenn Sie eine stabile öffentliche IP-Adresse und ein höheres Maß an Verfügbarkeit für Ihren Service benötigen, sollten Sie Ihre App über einen [Netzausgleichsfunktions- (NLB-) Service](/docs/containers?topic=containers-loadbalancer) oder über [Ingress](/docs/containers?topic=containers-ingress) verfügbar machen.
{: note}

<br />


## Zugriff auf eine App mithilfe eines NodePort-Service aktivieren
{: #nodeport_config}

Sie können Ihre App als einen Kubernetes-NodePort-Service für kostenlose Cluster oder Standardcluster zugänglich machen.
{:shortdesc}

Wenn bisher keine App bereitsteht, können Sie eine Kubernetes-Beispielapp namens [Guestbook ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml) verwenden.

1.  Definieren Sie in der Konfigurationsdatei für Ihre App einen Abschnitt vom Typ [Service ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://kubernetes.io/docs/concepts/services-networking/service/).

    Für das Beispiel 'Guestbook' ist in der Konfigurationsdatei ein Front-End-Serviceabschnitt vorhanden. Um die App 'Guestbook' extern verfügbar zu machen, müssen Sie dem Front-End-Serviceabschnitt den Typ 'NodePort' und für 'nodePort' eine Portnummer aus dem Bereich 30000 bis 32767 hinzufügen.
    {: tip}

    Beispiel:

    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: <mein_nodeport-service>
      labels:
        <mein_bezeichnungsschlüssel>: <mein_bezeichnungswert>
    spec:
      selector:
        <mein_selektorschlüssel>: <mein_selektorwert>
      type: NodePort
      ports:
       - port: <8081>
         # nodePort: <31514>

    ```
    {: codeblock}

    <table>
    <caption>Erklärung der Komponenten des NodePort-Service</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Ideensymbol"/> Erklärung der Komponenten des NodePort-Serviceabschnitts</th>
    </thead>
    <tbody>
    <tr>
    <td><code>metadata.name</code></td>
    <td>Ersetzen Sie <code><em>&lt;mein_nodeport-service&gt;</em></code> durch einen Namen für Ihren NodePort-Service.<p>Erfahren Sie mehr über das [Sichern der persönlichen Daten](/docs/containers?topic=containers-security#pi) bei der Arbeit mit Kubernetes-Ressourcen.</p></td>
    </tr>
    <tr>
    <td><code>metadata.labels</code></td>
    <td>Ersetzen Sie <code><em>&lt;mein_bezeichnungsschlüssel&gt;</em></code> und <code><em>&lt;mein_bezeichnungswert&gt;</em></code> durch die Bezeichnung, die Sie für Ihren Service verwenden möchten.</td>
    </tr>
    <tr>
      <td><code>spec.selector</code></td>
      <td>Ersetzen Sie <code><em>&lt;mein_selektorschlüssel&gt;</em></code> und <code><em>&lt;mein_selektorwert&gt;</em></code> durch das Schlüssel/Wert-Paar, das Sie im Abschnitt <code>spec.template.metadata.labels</code> Ihrer YAML-Bereitstellungsdatei verwendet haben. Wenn Sie den Service der Bereitstellung zuordnen möchten, muss der Selektor mit den Bereitstellungsbeschriftungen übereinstimmen.
      </tr>
    <tr>
    <td><code>ports.port</code></td>
    <td>Ersetzen Sie <code><em>&lt;8081&gt;</em></code> durch den Port, an dem Ihr Service empfangsbereit ist. </td>
     </tr>
     <tr>
     <td><code>ports.nodePort</code></td>
     <td>Optional: Ersetzen Sie <code><em>&lt;31514&gt;</em></code> durch eine Knotenportnummer aus dem Bereich 30000 bis 32767. Geben Sie für 'NodePort' keine Portnummer an, die bereits von einem anderen Service verwendet wird. Wenn manuell keine Knotenportnummer festgelegt wird, so erfolgt die Zuweisung automatisch nach dem Zufallsprinzip.<br><br>Wenn Sie für 'NodePort' eine Portnummer festlegen wollen und ermitteln möchten, welche Knotenportnummern bereits belegt sind, führen Sie den folgenden Befehl aus: <pre class="pre"><code>kubectl get svc</code></pre><p>Alle bereits belegten Knotenportnummern werden unter dem Feld **Ports** angezeigt.</p></td>
     </tr>
     </tbody></table>

2.  Speichern Sie die aktualisierte Konfigurationsdatei.

3.  Wiederholen Sie diese Schritte, um einen NodePort-Service für jede App zu erstellen, die Sie im Internet zugänglich machen wollen.

**Womit möchten Sie fortfahren?**

Bei Bereitstellung der App können Sie mithilfe der öffentlichen IP-Adresse jedes beliebigen Workerknotens und der für 'NodePort' festgelegten Portnummer die öffentliche URL erzeugen, mit der in einem Browser auf die App zugegriffen werden kann. Wenn Ihre Workerknoten nur mit einem privaten VLAN verbunden sind, wurde ein privater NodePort-Service erstellt, auf den über die private IP-Adresse des Warteschlangen-Workerknotens zugegriffen werden kann.

1.  Rufen Sie die öffentliche IP-Adresse für einen Workerknoten im Cluster ab. Wenn Sie auf den Workerknoten in einem privaten Netz zugreifen möchten, rufen Sie stattdessen die private IP-Adresse ab.

    ```
    ibmcloud ks workers --cluster <clustername>
    ```
    {: pre}

    Ausgabe:

    ```
    ID                                                Public IP   Private IP    Size     State    Status
    prod-dal10-pa215dcf5bbc0844a990fa6b0fcdbff286-w1  192.0.2.23  10.100.10.10  u3c.2x4  normal   Ready
    prod-dal10-pa215dcf5bbc0844a990fa6b0fcdbff286-w2  192.0.2.27  10.100.10.15  u3c.2x4  normal   Ready
    ```
    {: screen}

2.  Falls für 'NodePort' nach dem Zufallsprinzip Portnummern zugewiesen wurden, ermitteln Sie eine Portnummer, die auf diese Art festgelegt wurde.

    ```
    kubectl describe service <servicename>
    ```
    {: pre}

    Ausgabe:

    ```
    Name:                   <servicename>
    Namespace:              default
    Labels:                 run=<bereitstellungsname>
    Selector:               run=<bereitstellungsname>
    Type:                   NodePort
    IP:                     10.10.10.8
    Port:                   <unset> 8080/TCP
    NodePort:               <unset> 30872/TCP
    Endpoints:              172.30.171.87:8080
    Session Affinity:       None
    No events.
    ```
    {: screen}

    In diesem Beispiel lautet die Portnummer für 'NodePort' `30872`.

    Wenn der Abschnitt zu Endpunkten (**Endpoints**) `<none>` anzeigt, überprüfen Sie den Selektorschlüssel (`<selectorkey>`) und den Selektorwert (`<selectorvalue>`), die Sie im Abschnitt `spec.selector` des NodePort-Service verwenden. Stellen Sie sicher, dass es sich hierbei um dasselbe _Schlüssel/Wert_-Paar handelt, das Sie im Abschnitt `spec.template.metadata.labels` der YAML-Bereitstellungsdatei verwendet haben.
    {: note}

3.  Bilden Sie die URL mit einer der IP-Adressen des Workerknotens und der Portnummer für 'NodePort'. Beispiel: `http://192.0.2.23:30872`
