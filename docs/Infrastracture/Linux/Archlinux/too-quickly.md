# systemd serviceが起動しないときの対処法 - too quickly

- [元になったページ](https://www.hiroom2.com/2017/02/18/linux-systemd%E3%81%AEstart-request-repeated-too-quickly-for-xxx-service/)

Archlinuxを使っている時に「start request repeated too quickly for lightdm.service」などと出力されてサービスが起動しない場合の対処法

systemctl reload

これを用いると、systemctl reloadと入力した際に

```
$ systemctl show -p CanReload named
CanReload=yes
```

と出力されたサービスはシェルスクリプトを書くことで10秒以内に何回でもリロードできる。以下はスクリプト。

```
bash=
sudo systemctl start lightdm
i=1
while : ; do
  echo ${i}
  i=$(expr ${i} + 1);
  sudo systemctl restart lightdm || break
done
```

systemctl reloadが使えない場合

```
$ systemctl show -p CanReload dhcpd
CanReload=no
```

この場合はリロードすることができない。この場合は、StartLimitBurst=0をsystemdファイルに追記することで再起動回数の判定を無効にできる。ただし、起動時に再起動を繰り返す恐れのあるサービス以外に限ります。起動しなくなるよ:)
以下でsystemdのファイルを確認します。

```
$ systemctl show -p FragmentPath dhcpcd
FragmentPath=/usr/lib/systemd/system/dhcpcd.service
```

\[Service\]のセクションにStartLimitBurst=0を設定します。

```
bash=
[Service]
 Type=notify
 ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

StartLimitBurst=0
##↑追記

 [Install]
 WantedBy=multi-user.target
```

つまり、lightdmのsystemdファイルにStartLimitBurst=0を追記すれば理論上は起動する。