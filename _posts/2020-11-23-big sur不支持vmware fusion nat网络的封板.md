当前版本的Macos Big Sur11.01不支持vmware fusion的nat网络模式，表现为无法连接/dev/vmnet8

连接外网：使用桥接模式，由于实验室网络管理原因，需要将虚拟机的mac地址与主机mac地址设置为相同，两者有相同的ip地址

ssh远程连接：使用仅主机模式，通过ssh连接管理虚拟机。

等待后续更新