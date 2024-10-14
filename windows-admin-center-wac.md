# Windows Admin Center (WAC)

## AD01 SETUP

Change IP Configuration

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Change hostname to AD01-Fatima

```
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "fatima.local"
Install-WindowsFeature DNS -IncludeManagementTools
Add-DnsServerResourceRecordA -CreatePtr -Name "FS01-fatima" -ZoneName "fatima.local" -AllowUpdateAny -IPv4Address "10.0.5.7"
Add-DnsServerResourceRecordPtr -Name "LAN" -ZoneName "5.0.10.in-addr.arpa" -AllowUpdateAny -PtrDomainName "AD01-fatima.fatima.local"
```

Adding users:

In Server Manager > Tools > Users and Computers > fatima.local > Users

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>Add the -ADM user to group Domain Admin</p></figcaption></figure>





## FS01:

Network Setup using sconfig

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Join fatima.local domain and change hostname to FS01-Fatima

Installing and Using WAC

> ```
> Invoke-WebRequest -Uri https://go.microsoft.com/fwlink/p/?linkid=2194936 -outfile windowsAdminCenter-xxx.msi
> ```

Go through all of the installer pop ups and keep default values



Navigate to: https://fs01-fatima.fatima.local to access WAC

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption><p>Added devices ad01 and wks01 with the +Add in top left</p></figcaption></figure>

\
Adding extensions: WAC > Settings > Extensions
----------------------------------------------

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption><p>Install the DNS and Active Directory extensions by right clicking</p></figcaption></figure>



## Adding GPO for WinRM

Create a new OU in AD for WKS

Create a GPO > Right Click Edit

Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Remote Management > WinRM Service and select ALLOW remote server management through WinRM and enable it. For the Pv4/6 filters do a \*



Computer Configuration > Policies > Windows Settings > Security Settings > System Services > Windows Remote Management check Define this policy setting and select Automatic



Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security select "Predefined" and scroll down to select Windows Remote Management



gpupdate /force on windows boxes and it will work.
