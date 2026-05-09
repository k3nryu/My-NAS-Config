# My-NAS-Config
NAS配置（Git + Ansible Vault + ZFS）设计/重建说明书

##  README.md —— 灾难恢复的“入口按钮”

这是我们 OS 崩了以后唯一需要打开的文件。

它应该回答：

我们要装什么 OS

我们要装哪些包

我们要跑哪一个命令

哪一步 绝对不能做错

👉 README ≠ 项目介绍，而是救命手册

---

## ansible/ —— 把“声明”变成现实

Ansible 在这个 repo 里只干一件事：

把 Git 里的状态，强制同步到系统

灾难恢复时你只做：

```
ansible-playbook ansible/site.yml --ask-vault-pass
```


它应该：

不依赖当前系统状态

可重复执行

不怕执行 10 次

---

## docs/ —— 这是你“未来不骂过去自己的保险”

这里不是给 `Ansible` 用的，是给 `人` 用的。

尤其重要的是：

architecture.md（为什么是 ZFS？为什么这样分 dataset？）

decisions/（当初为什么这样设计 ACL）

👉 你半年后一定会忘

---


## scripts/ —— 防止“无意识作死”

这些脚本不是必须，但非常值：

diff 当前系统 vs Git

OS 升级前自检

升级后验证关键点

这是你从“能恢复”迈向“稳”的一步。

---

灾难恢复时，我们真正要做的只有这些

假设：OS 盘已坏，ZFS 数据盘完好

1. 重装最小 OS（别碰数据盘）
2. 安装依赖
```
dnf install -y git ansible zfs
```

3. 拉 repo
```
git clone https://github.com/k3nryu/My-NAS-Config.git
cd My-NAS-Config
```

4. 先导入已有 ZFS pool
```
zpool import
zpool import tank
zpool status tank
```

5. 跑 Ansible
```
ansible-playbook ansible/site.yml --ask-vault-pass
```

👉 恢复结束
这就是这个 repo 的 KPI。

更完整的恢复文档：

- `docs/inventory.md`
- `docs/disaster-recovery.md`
- `docs/recovery-checklist.md`
- `docs/restore-drill.md`
