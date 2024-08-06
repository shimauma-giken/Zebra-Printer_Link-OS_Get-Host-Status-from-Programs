# Zebra-Printer_Link-OS_Get Host Status from Programs
# アプリケーションからプリンタのホストステータスを取得する方法

<img width="500" src="https://images.unsplash.com/photo-1526498460520-4c246339dccb?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D">

ホストのアプリケーションからステータスを取得する代表的な例を紹介する。
各取得方法に長短があるので、見極めた上で運用に適したものを利用すること。

[参考: Zebra プリンタからRFID履歴/プリンタステータスを取得するコード例 C#編](https://github.com/shimauma-giken/Zebra-Printer_C-Sharp_Get-Printer-Status_RFID-Logs )
[参考: Git-Hub: Android 11 向けZebraプリンタ用サンプルコード](https://github.com/shimauma-giken/Zebra-Printer_Android-Sample-Code_Print_Pause_Cancel)

</br>

## ■ SDK: Class PrinterStatus

プログラムからSDK経由でコールが可能。
PrinterStatus Classを用いて、限られたステータス情報を取得可能。
開発コストは低いが、レスポンス性は高くない。

[Techdocs: Class PrinterStatus](https://techdocs.zebra.com/link-os/2-14/pc/content/com/zebra/sdk/printer/printerstatus)

```java
Connection connection = new TcpConnection("192.168.1.100", TcpConnection.DEFAULT_ZPL_TCP_PORT);
         try {
             connection.open();
             ZebraPrinter printer = ZebraPrinterFactory.getInstance(connection);
 
             PrinterStatus printerStatus = printer.getCurrentStatus();
             if (printerStatus.isReadyToPrint) {
                 System.out.println("Ready To Print");
             } else if (printerStatus.isPaused) {
                 System.out.println("Cannot Print because the printer is paused.");
             } else if (printerStatus.isHeadOpen) {
                 System.out.println("Cannot Print because the printer head is open.");
             } else if (printerStatus.isPaperOut) {
                 System.out.println("Cannot Print because the paper is out.");
             } else {
                 System.out.println("Cannot Print.");
             }
         } catch (ConnectionException e) {
             e.printStackTrace();
         } catch (ZebraPrinterLanguageUnknownException e) {
             e.printStackTrace();
         } finally {
             connection.close();
         }
```

- Eg.Output Data
```
Cannot Print because the paper is out.
```


</br>
</br>

## ■ SDK: Class SGD + "~HS"

プログラムからSDK経由でコールが可能。
PrinterStatus Classより、詳細な情報を取得可能。
比較的レスポンスが早いため、反応性が求められる現場で多様される。

[Techdocs: Class SGD](https://techdocs.zebra.com/link-os/2-14/pc/content/com/zebra/sdk/printer/sgd)


```java
try {
             printerConnection.open();
             // Get an SGD, using the status channel
             String status = SGD.GET("device_host_status", printerConnection);
             System.out.println("SGD status is " + status);
             // Close the connection
             printerConnection.close();
         } catch (ConnectionException e) {
             e.printStackTrace();
         } finally {
             printerConnection.close();
         }
```

- Eg.Output Data
```
"159,0,0,0364,000,0,0,0,000,0,0,0
000,0,0,0,0,1,4,0,00000000,1,000
0000,0"
```

</br>
</br>

## ■ SDK: Class SnmpPrinter

OIDを用いて詳細なステータス・パラメータ情報を取得可能。
ネットワーク接続の場合に限られるが、比較的レスポンス性が高く、詳細なステータス情報が取得できるのが強み。
SNMPの詳細は 別頁を参照。

```java
An instance of an SNMP only Zebra printer. The printer does not make a raw port connection.
package test.zebra.sdk.printer.examples;
 
 import com.zebra.sdk.printer.SnmpException;
 import com.zebra.sdk.printer.SnmpPrinter;
 
 public class SnmpPrinterExample {
 
     public static void main(String[] args) {
         try {
             SnmpPrinter myPrinter = new SnmpPrinter("1.2.3.4", "getCommunityName", "setCommunityName");
 
             String printAdjOid = "1.3.6.1.4.1.10642.6.4.0";
             String printAdjFromSnmp = myPrinter.getOidValue(printAdjOid);
             System.out.println("printAdj = " + printAdjFromSnmp);
 
             int newprintAdj = Integer.parseInt(printAdjFromSnmp) + 1;
             myPrinter.setOidValue(printAdjOid, newprintAdj);
 
             String friendlyNameOid = "1.3.6.1.4.1.10642.20.3.5.0";
             String friendlyNameFromSnmp = myPrinter.getOidValue(friendlyNameOid);
             System.out.println("friendlyName = " + friendlyNameFromSnmp);
 
             String newfriendlyName = "AwesomePrinter";
             myPrinter.setOidValue(friendlyNameOid, newfriendlyName);
 
         } catch (SnmpException e) {
             e.printStackTrace();
         }
     }
 }
```