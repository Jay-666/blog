>越麻烦越安全，越简单就越危险

1. 注意权限分离（Linux系统权限、数据库权限不要掌握在同一部门）
2. 权限在满足使用的情况下，最小优先
3. 减少使用root用户，尽量用“普通用户+sudo权限”进行日常操作，比如用户管理
4. 重要系统文件，如：/etc/passwd、/etc/shadow、/etc/fstab、/etc/sudoers 等，日常建议使用chattr锁定，需要操作的时候再打开
5. 使用脚本检测系统中新增的SUID、SGID文件
6. 可以利用工具（如chkrootkit等）检测rootkit脚本（杀毒）
7. 开启SSH服务秘钥对登录，修改SSH服务默认端口