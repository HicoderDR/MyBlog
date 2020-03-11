.. post::Dec 19,2019
    :tags:python
    :category:python
    :author:HicoderDR

Paramiko：Python+ssh做快乐网管
#############################################################
本篇博客记载在配置ACM—ICPC赛制下比赛机房环境时使用的手段

机房环境
****************************
一百多台电脑使用保护卡分发的Linux-manjaro系统，配置好openssh。
（因为是一台机器的系统分发到所有机器，所以IP和ssh都一摸一样）

库的安装
*************************
**pip install paramiko**

分发后自动根据MAC地址配置IP
**********************************
首先在机房安装的其他系统下，使用IP扫描工具，获取到所有电脑的IP地址和对面的网卡MAC地址。

并将其保存下来，在python中读取为dictionary，之后远程调用命令行执行修改IP操作即可。

ssh网管操作
******************************
::

    import time
    import paramiko
    import threading

    class Client:
        def __init__(self, ip):
            self.ssh = paramiko.SSHClient()
            self.ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            self.ssh.connect(hostname=ip,port=22,username='manjaro',password='password')
            self.sftp = paramiko.SFTPClient.from_transport(self.ssh.get_transport())
            self.sftp = self.ssh.open_sftp()
        
        def create_account(self, username, password):
            stdin, stdout, stderr = self.ssh.exec_command('sudo adduser {} --force-badname'.format(username))
            stdin.write('{}\n{}\n\n\n\n\n\n\ny\n'.format(password, password))
            stdin.flush()
            result = stdout.read()
            if not result:
                result = stderr.read()
            print(result.decode())
            return self
        
        def change_password(self, username, password):
            stdin, stdout, stderr = self.ssh.exec_command('passwd {}'.format(username))
            stdin.write('{}\n{}\n'.format(password, password))
            stdin.flush()
            result = stdout.read()
            if not result:
                result = stderr.read()
            print(result.decode())
            return self
        
        def exec_cmd(self, cmd):
            stdin, stdout, stderr = self.ssh.exec_command(cmd)
            result = stdout.read()
            if not result:
                result = stderr.read()
            print(result.decode())
            return self
        
        def copy_from_remote(self, local_file, remote_file):
            self.sftp.get(local_file, remote_file)
            return self
        
        def copy_to_remote(self, local_file, remote_file):
            self.sftp.put(local_file, remote_file)
            return self
        
        def list_dir(self):
            print(self.sftp.listdir())

        def shutdown():
		    self.ssh.exec_command('shutdown')
    
    def work(i,cmd):
        ip = '172.xxx.xxx.'+ str(i)
        Client(ip).exec_cmd(cmd)
        print(ip + " worked!")

    if __name__ == '__main__':
        threads = []
        for i in range(1,100):
            threads.append(threading.Thread(target=work,args=(i,'reboot')))
        for t in threads:
            t.start()
            time.sleep(0.5)
        for t in threads:
            t.join()

