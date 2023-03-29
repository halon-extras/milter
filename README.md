# Milter

Implementation of the Milter protocol v6 for EOD script.
Integration tested with clamav-milter, Cloudmark Authority Milter and SAFE/25 (inet socket).

## Milter(opts, senderip, senderport, senderptr, senderhelo, sender, recipients, transactionid, mail)

Send message to milter and apply changes to the contents in accordance with the milter response.

**Parameters**

- opts (`array`) - an options array
- senderip (`string`) - remoteip
- senderport (`number`) - remoteport
- senderptr (`string`) - remoteptr
- senderhelo (`string`) - helo
- sender (`string`) - envelope-from
- recipients (`array`) - envelope-to
- transactionid (`string`) - transaction id
- mail (`EODMailMessage`) - An instance of the mail message

**Returns**: class object.

The following options are available in the **opts** array.

- host (`string`) - IP address of a Milter service.
- port (`number`) - TCP port.
- timeout (`number`) - Timeout in seconds. The default is 5 seconds.
- timeout_eod (`number`) - Timeout in seconds. The default is 5 seconds.


## getHeaderValue()

Return header value from the header field if it exists

**Returns**:  header value (`string`) or none


## getAction()

Return what action caller needs to take

**Returns**: action (`string`) : none, discard, reject, setreply, quarantine, tempfail

## getReply()

Return reply code and reason for setreply for Reject() and Defer()

**Returns**: array with reason and reply_codes

["reason" => `reason`, "reply_codes" => ["code" => `smtpcode`, "enhanced" => `enhanced`]];

- reason (`string`) - reason
- reply_codes (`array`) - reply_codes for Reject() and Defer()
- code (`number`) - reply smtpcode such as 550
- enhanced (`array`) - reply dsn code such as 5.7.1 as an array [5,7,1]

# example

```
import { Milter } from "milter/milter.hsl";

$senderip = $connection["remoteip"];
$senderport = $connection["remoteport"];
$senderptr = $connection["remoteptr"];
$senderhelo = $connection["helo"]["host"];
$sender = $transaction["sender"];
$recipients = [];
foreach($transaction["recipients"] as $r) {
  if($r["recipient"] != "") {
    $recipients[] = $r["recipient"];
  }
}
$transactionid = $transaction["id"];

$opts = ["host" => "127.0.0.1", "port" => 2704, "timeout_eod" => 15];
$mc = Milter($opts, $senderip, $senderport, $senderptr, $senderhelo, $sender, $recipients, $transactionid, $arguments["mail"]);

echo $mc->getHeaderValue("X-CMAE-Score");  // for Cloudmark Authority Milter
$action = $mc->getAction();
if($action == "discard") {
    // return 250 and not put in queue
    Accept();
}
else if($action == "tempfail") {
    Defer("Message deferred.");
}
else if($action == "reject") {
    Reject("Message rejected.");
}
else if($action == "setreply") {
    $r = $mc->getReply();
    Reject($r["reason"], ["reply_codes" => $r["reply_codes"]]);
}
else if($action == "quarantine") {
    // do something
}

// put in queue
foreach ($transaction["recipients"] as $r) {
  $sender = $transaction["senderaddress"];
  $recipient = $r["address"];

  $arguments["mail"]->queue($transaction["senderaddress"], $r["address"], $r["transportid"]);
}

Accept();
```
