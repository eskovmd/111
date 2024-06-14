- SRV-L
    1. apt-get update && apt-get install bind bind-utils -y
    2. systemctl enable --now bind
    3. vim /etc/bind/options.conf
        1. что должно быть в options:
        listen-on { any; };
        forwarders { 94.232.137.104; }; (ip адрес DNS в задании)
        dnssec-validation no;
        recursion yes;
        allow-query { any; };
        allow-recursion { any; };

        ![21](https://github.com/eskovmd/111/assets/172771001/98705923-889e-4060-8c78-a54ec9f1c245)

        4. vim /etc/bind/local.conf
    1. добавляем после слов Add other zones here:
    zone "au.team" {
                    type master;
                    file "au.team";
                    allow-transfer {20.20.20.100;};
    };
    zone "10.10.10.in-addr.arpa" {
                    type master;
                    file "left.reverse";
                    allow-transfer {20.20.20.100;};
    };
    zone "20.20.20. in-addr.arpa" {
                    type master;
                    file "right.reverse";
                    allow-transfer {20.20.20.100;};
    };
    zone "35.35.35. in-addr.arpa" {
                    type master;
                    file "cli.reverse";
                    allow-transfer {20.20.20.100;};

                    ![22](https://github.com/eskovmd/111/assets/172771001/e4ae1b26-84ae-492d-a5be-64453a89fe7b)

                    cd /etc/bind/zone/
cp localhost au.team
vim au.team
заменяем localhost. на au.team.  и root.localhost. на root.au.team.
редачим зоны, должны получиться такие зоны, пишем через табуляцию (Tab), а не пробелы:
@        IN     NS    au.team.
@        IN     NS    10.10.10.100
isp      IN     A     100.100.100.1
rtr-l    IN     A     10.10.10.1
rtr-r    IN     A     20.20.20.1
web-l    IN     A     10.10.10.110
web-r    IN     A     10.10.10.100
cli      IN     A     35.35.35.10
dns      IN     CNAME srv-l
ntp      IN     CNAME isp
mediawiki       IN    CNAME    web-l

![23](https://github.com/eskovmd/111/assets/172771001/369e1af3-324d-429e-bbc1-3d8a04b2ea1f)

cp localhost right.reverse
vim right.reverse
заменяем localhost. на 20.20.20.in-addr.arpa. и root.localhost. на root.20.20.20.in-addr.arpa.
редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
@        IN     NS    au.team.
@        IN     NS    10.10.10.100
1        PTR    rtr-r.au.team.
100      PTR    web-r.au.team.

cp right.reverse left.reverse
vim left.reverse
заменяем 20.20.20.in-addr.arpa. на 10.10.10.in-addr.arpa. и root.localhost. на root.10.10.10.in-addr.arpa.
редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
@        IN     NS    au.team.
@        IN     NS    10.10.10.100
1        PTR    rtr-l.au.team.
100      PTR    srv-l.au.team.
110      PTR    web-l.au.team.


cp right.reverse cli.reverse
vim cli.reverse
заменяем 10.10.10.in-addr.arpa. на 35.35.35.in-addr.arpa. и root.localhost. на root.35.35.35.in-addr.arpa.
редачим зоны, должны получиться такие зоны пишем через табуляцию (Tab), а не пробелы:
@        IN     NS    au.team.
@        IN     NS    10.10.10.100
1        PTR    rtr-l.au.team.
100      PTR    srv-l.au.team.
110      PTR    web-l.au.team.

14. chmod 777 au.team
15. chmod 777 right.reverse
16. chmod 777 left.reverse
17. chmod 777 cli.reverse
18. systemctl restart bind
19. echo  “nameserver 127.0.0.1” > /etc/net/ifaces/enp0s8/resolv.conf
20. vim /etc/resolv.conf
    1. должен быть указан только один nameserver 127.0.0.1

    ![24](https://github.com/eskovmd/111/assets/172771001/ea2d962d-4c55-42c5-aa91-7619560dd9d3)
если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь



- WEB-R
    1. apt-get update && apt-get install bind bind-utils -y
    2. systemctl enable --now bind
    3. vim /etc/bind/options.conf
        1. что должно быть в options:
        listen-on { any; };
        forwarders { 10.10.10.100; };
        dnssec-validation no;
        recursion yes;
        allow-query { any; };
        allow-recursion { any; };

       4. vim /etc/bind/local.conf
    1. добавляем после слов Add other zones here:
    zone "au.team" {
                    type slave;
                    file "slave/au.team";
                    masters {10.10.10.100;};
    };
    zone "10.10.10.in-addr.arpa" {
                    type slave;
                    file "slave/left.reverse";
                    masters {10.10.10.100;};
    };
    zone "20.20.20. in-addr.arpa" {
                    type slave;
                    file "slave/right. reverse";
                    masters {10.10.10.100;};
    };
    zone "35.35.35. in-addr.arpa" {
                    type slave;
                    file "slave/cli. reverse";
                    masters {10.10.10.100;};
    };

5. chown named:named /var/lib/bind/zone/slave/
6. chown named:named /etc/bind/zone/slave/
7. systemctl restart bind
8. echo  “nameserver 127.0.0.1” > /etc/net/ifaces/enp0s3/resolv.conf
9. vim /etc/resolv.conf
    1. должен быть указан только один nameserver 127.0.0.1

2. если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь

После того как DNS сервера настроены, надо указать на каждой машине,
кроме самих DNS серверов, на них мы уже себя указали, 
указать в качестве DNS сервера наши сервера.

  - CLI
    1. echo  “nameserver 100.100.100.10” > /etc/net/ifaces/enp0s3/resolv.conf
    2. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 100.100.100.10

        ![25](https://github.com/eskovmd/111/assets/172771001/d952e22d-5399-4343-b66d-a65c77f87521)
        если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь

   - ISP
    1. echo  “nameserver 100.100.100.10” > /etc/net/ifaces/enp0s8/resolv.conf
    2. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 100.100.100.10

       - RTR-L
    1. echo  “nameserver 10.10.10.100” > /etc/net/ifaces/enp0s8/resolv.conf
    2. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100

          ![26](https://github.com/eskovmd/111/assets/172771001/5ca05075-1fb2-4daa-9ba4-60767b6eb3d8)
        2. если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь
- RTR-R
    1. echo  “nameserver 20.20.20.100” > /etc/net/ifaces/enp0s8/resolv.conf
    2. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 20.20.20.100

        
        2. если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь
      
- WEB-L
    1. echo  “nameserver 10.10.10.100” > /etc/net/ifaces/enp0s3/resolv.conf
    2. vim /etc/resolv.conf
        1. должен быть указан только один nameserver 10.10.10.100
              
        2. если пропал интернет, добавляем nameserver 94.232.137.104 или nameserver 8.8.8.8, может помочь       
