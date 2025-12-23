# ZFS snapshot 策略说明
利用sanoid自动备份，激动修剪snapshot
> https://github.com/jimsalterjrs/sanoid.git

备份策略为使用sanoid的样本，即最近每36小时各一份，最近每30天各一份，最近每3个月各一份。
> /etc/sanoid/sanoid.conf

```
[tank]
	use_template = production
	recursive = zfs
[template_production]
	frequently = 0
	hourly = 36
	daily = 30
	monthly = 3
	yearly = 0
	autosnap = yes
	autoprune = yes
```
