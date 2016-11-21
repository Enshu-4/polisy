#ポリシー

INSIDEPC="192.168.20.1"
IP="192.168.1.33"
ALL="0.0.0.0/0"
INT="192.168.10.1"
DMZ="192.168.10.0/24"
WWW="192.168.10.2:80"
SSH="192.168.10.3:22"

##EXT->DMZ
外部PCからSSHはできない
外部PCからWWWに接続できる

##DMZ->EXT                                                                    
WWWまたはSSHから外部に接続できる

##INT->DMZ                                                                   
内部PCからWWWに接続できる
内部PCからSSHに接続できる

##DMZ->INT                                                                    
WWWから内部PCに接続できない
SSHから内部PCに接続できない

##EXT->INT                                               
外部PCから内部PCに接続できない

##INT->EXT        
内部PCから外部PCに接続できない

File Edit Options Buffers Tools Sh-Script Help                                  
DMZ="192.168.10.0/24"
WWW="192.168.10.2:80"
SSH="192.168.10.3:22"
sudo iptables -A PREROUTING -t nat -p tcp -d $IP --dport 80 -i eth0 -j DNAT --t\
o $WWW
sudo iptables -A PREROUTING -t nat -p tcp -d $IP --dport 22 -i eth0 -j DNAT --t\
o $SSH

#ユーザーチェインを定義                                                         
sudo iptables -N ext-dmz
sudo iptables -N dmz-ext
#sudo iptables -N int-dmz                                                       
#sudo iptables -N dmz-int sudo iptables -N ext-int                              
sudo iptables -N int-ext

#FORWARD                                                                        
sudo iptables -A FORWARD -s $EXT -d $DMZ -o eth0 -j ext-dmz
sudo iptables -A FORWARD -s $DMZ -d $EXT -o eth0 -j dmz-ext
#iptables -A FORWARD -s $INT -d $DMZ -o eth1 -j int-dmz                         
#iptables -A FORWARD -s $DMZ -d $INT -o eth1 -j dmz-int                         
sudo iptables -A FORWARD -s $EXT -d $INT -o eth0 -j ext-int
sudo iptables -A FORWARD -s $INT -d $EXT -o eth0 -j int-ext

#WWWの設定                                                                      
sudo iptables -A ext-dmz -p tcp --dport 80 -j ACCEPT
sudo iptables -A dmz-ext -p tcp --sport 80 -j ACCEPT

sudo iptables -A FORWARD -j DROP

#ssh によるアクセスを拒否する                                                   
#sudo iptables -A INPUT -p tcp -s $OUTSIDEPC --dport ssh -i eth0 -j DROP        

#ipマスカレード                                                                 
sudo iptables -A POSTROUTING -t nat -j MASQUERADE -o eth0
