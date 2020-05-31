

SELinux管理

SELinux是由美国国家安全局（NSA）开发的，整合在Linux内核中，针对特定的进程与指定的文件资源进行权限控制的系统。

即使你是root用户，也必须遵守SELinux的规则，才能正确访问正确的资源，这样可以有效的防止root用户的误操作（当然，root用户可以修改SELinux的规则）。