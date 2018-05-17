# linux_howto
# Linux下通过rdesktop连接Windows远程桌面

一、工具：rdesktop

尝试了各种工具，但是在连接Windows远程桌面时总会出现各种不能连接的异常情况。

最后，决定尝试搞定rdesktop。



二、安装：

- Debian（Ubuntu）系统下执行：
```
    $ sudo apt-get install rdesktop
```
三、配置和使用

使用如下命令即可直接连接远程桌面
```
    $ rdesktop -f IP (windowsip地址)
```
- -f参数默认全屏打开，使用Ctrl + Alt + Enter可以退出全屏模式。
  但是实际情况是由于Windows安全性要求

使用rdesktop或者其它linux rdp客户端时，总会出现连接失败的情况，例如rdesktop：
```
    $ rdesktop IP
    ERROR: CredSSP: Initialize failed, do you have correct kerberos tgt initialized ?
    Failed to connect, CredSSP required by server.
```
四、配置kerberos认证解决rdesktop连接失败的问题

- Debian（Ubuntu）系统下执行：
```
    sudo apt-get install krb5-user libpam-krb5 libpam-ccreds krb5-doc nss-updatedb
```
- 安装过程配置realm等。如果对配置过程有疑问，可以在安装后编辑/etc/krb5.conf

    $ sudo vi /etc/krb5.conf

- 修改或添加以下关键配置信息
```
    [libdefaults]
    	dns_lookup_kdc = true
    	dns_lookup_realm = false
    	default_realm = YOURAD.COM
    	ticket_lifetime = 240000
    
    [realms]
    	yourname@yourad.com = {
    		kdc = pdc.YOURAD.com
    		kdc = dc02.YOURAD.com
    		admin_server = dc02.YOURAD.com
    		default_domain = YOURAD.COM
    
    [domain_realm]
    	.yourad.com = YOURAD.COM
    	yourad.com = YOURAD.COM
```

- 确认无误后，可尝试执行以下信息
```
    $ kinit yourname
```
如未出现错误、并且输入密码后正常，可验证ticket是否获取成功
```
    $ klist
```
再次尝试使用rdesktop IP 连接，即可发现已经一切正常

参考：rdesktop参数
```
    rdesktop: A Remote Desktop Protocol client.
    Version 1.8.3. Copyright (C) 1999-2011 Matthew Chapman et al.
    See http://www.rdesktop.org/ for more information.
    
    Usage: rdesktop [options] server[:port]
       -u: user name
       -d: domain
       -s: shell / seamless application to start remotly
       -c: working directory
       -p: password (- to prompt)
       -n: client hostname
       -k: keyboard layout on server (en-us, de, sv, etc.)
       -g: desktop geometry (WxH)
       -i: enables smartcard authentication, password is used as pin
       -f: full-screen mode
       -b: force bitmap updates
       -L: local codepage
       -A: path to SeamlessRDP shell, this enables SeamlessRDP mode
       -B: use BackingStore of X-server (if available)
       -e: disable encryption (French TS)
       -E: disable encryption from client to server
       -m: do not send motion events
       -C: use private colour map
       -D: hide window manager decorations
       -K: keep window manager key bindings
       -S: caption button size (single application mode)
       -T: window title
       -t: disable use of remote ctrl
       -N: enable numlock syncronization
       -X: embed into another window with a given id.
       -a: connection colour depth
       -z: enable rdp compression
       -x: RDP5 experience (m[odem 28.8], b[roadband], l[an] or hex nr.)
       -P: use persistent bitmap caching
       -r: enable specified device redirection (this flag can be repeated)
             '-r comport:COM1=/dev/ttyS0': enable serial redirection of /dev/ttyS0 to COM1
                 or      COM1=/dev/ttyS0,COM2=/dev/ttyS1
             '-r disk:floppy=/mnt/floppy': enable redirection of /mnt/floppy to 'floppy' share
                 or   'floppy=/mnt/floppy,cdrom=/mnt/cdrom'
             '-r clientname=<client name>': Set the client name displayed
                 for redirected disks
             '-r lptport:LPT1=/dev/lp0': enable parallel redirection of /dev/lp0 to LPT1
                 or      LPT1=/dev/lp0,LPT2=/dev/lp1
             '-r printer:mydeskjet': enable printer redirection
                 or      mydeskjet="HP LaserJet IIIP" to enter server driver as well
             '-r sound:[local[:driver[:device]]|off|remote]': enable sound redirection
                         remote would leave sound on server
                         available drivers for 'local':
                         alsa:	ALSA output driver, default device: default
             '-r clipboard:[off|PRIMARYCLIPBOARD|CLIPBOARD]': enable clipboard
                          redirection.
                          'PRIMARYCLIPBOARD' looks at both PRIMARY and CLIPBOARD
                          when sending data to server.
                          'CLIPBOARD' looks at only CLIPBOARD.
             '-r scard[:"Scard Name"="Alias Name[;Vendor Name]"[,...]]
              example: -r scard:"eToken PRO 00 00"="AKS ifdh 0"
                       "eToken PRO 00 00" -> Device in Linux/Unix enviroment
                       "AKS ifdh 0"       -> Device shown in Windows enviroment 
              example: -r scard:"eToken PRO 00 00"="AKS ifdh 0;AKS"
                       "eToken PRO 00 00" -> Device in Linux/Unix enviroment
                       "AKS ifdh 0"       -> Device shown in Windows enviroment 
                       "AKS"              -> Device vendor name                 
       -0: attach to console
       -4: use RDP version 4
       -5: use RDP version 5 (default)
       -o: name=value: Adds an additional option to rdesktop.
               sc-csp-name        Specifies the Crypto Service Provider name which
                                  is used to authenticate the user by smartcard
               sc-container-name  Specifies the container name, this is usally the username
               sc-reader-name     Smartcard reader name to use
               sc-card-name       Specifies the card name of the smartcard to use
```
