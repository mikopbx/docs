# Getting logs using the tcpdump application

**tcpdump** is a Linux utility for capturing and analyzing network traffic. It allows you to monitor packets passing through a network interface in real time and save them for further analysis.

## Capturing Logs with the tcpdump Application

1. Connect to your PBX via SSH (instructions [here](connecting-to-a-pbx-using-ssh/connecting-to-a-pbx-using-an-ssh-client.md)).
2. Run the following command:

```bash
tcpdump -i eth0 -n -s 0 -vvv -w /tmp/capturefilename.pcap
```

<figure><img src="../../.gitbook/assets/firstOutput.png" alt=""><figcaption><p>Command in SSH connection</p></figcaption></figure>

3. Reproduce the issue by making a phone call. After the call, press **CTRL + C** in the SSH console to stop **tcpdump**.

<figure><img src="../../.gitbook/assets/secondOutput.png" alt=""><figcaption><p>Command in SSH connection</p></figcaption></figure>

4. Connect to the PBX using WinSCP (instructions [here](connecting-to-a-pbx-using-winscp.md)).
5. Send the log file **/tmp/capturefilename.pcap** to technical support.

<figure><img src="../../.gitbook/assets/fileWithLogs.png" alt=""><figcaption><p>File with logs</p></figcaption></figure>
