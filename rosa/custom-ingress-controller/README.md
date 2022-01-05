What I did was: 
1. spun up the new-ingress-controller to see its ocnfiguration 
   1. it updated the worker-sg inbound rules with the following configuration 
      1. 31709 - 0.0.0.0/0
      2. 31647 - 0.0.0.0/0
      3. 30897 - 10.0.0.0/17
   2. it created 2 target groups 
      1. tcp 31709, type instance, health check: http /healthz 30897
      2. tcp 31647, type instance, health check http /healthz 30897 
   3. nlb listeners: 
      1. 80 --> 31647
      2. 443 --> 31709
   4. created dns records
      1. public zone:
         1. A record *.extra.df-rosa-1.2wi6.p1.openshiftapps.com --> kube nlb 
      2. private zone: 
         1. A record *.extra.df-rosa-1.2wi6.p1.openshiftapps.com --> kube nlb 
   
2. in aws manually created a nlb (custom-nlb)
   1. depended on inbound rules described above to be in place, if using ELB those rules would not be there possibly...  
   2. nlb listeners --> set to match the ingress controller nlb 
3. manually created 2 custom target groups matching the ingress-controller's nlb ones 
   1. the target groups passed the health checks 
4. in the public dns zone (2wi6.p1.openshiftapps.com)
   1. created a Weighted A record  *.extra.df-rosa-1.2wi6.p1.openshiftapps.com --> custom nlb, weight 50
   2. updated the Simple A record to be weighted  *.extra.df-rosa-1.2wi6.p1.openshiftapps.com --> kube nlb, weight 50 

played with the weights, and used dig to retrieve records: $ dig *.extra.df-rosa-1.2wi6.p1.openshiftapps.com
```
    freese@localhost custom-ingress-controller]$ dig *.extra.df-rosa-1.2wi6.p1.openshiftapps.com

    ; <<>> DiG 9.11.13-RedHat-9.11.13-2.fc30 <<>> *.extra.df-rosa-1.2wi6.p1.openshiftapps.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17023
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4096
    ;; QUESTION SECTION:
    ;*.extra.df-rosa-1.2wi6.p1.openshiftapps.com. IN        A

    ;; ANSWER SECTION:
    *.extra.df-rosa-1.2wi6.p1.openshiftapps.com. 28 IN A 3.23.146.178

    ;; Query time: 0 msec
    ;; SERVER: 127.0.0.1#53(127.0.0.1)
    ;; WHEN: Thu Dec 23 12:43:05 EST 2021
    ;; MSG SIZE  rcvd: 88
```
```
dfreese@localhost custom-ingress-controller]$ dig *.extra.df-rosa-1.2wi6.p1.openshiftapps.com

; <<>> DiG 9.11.13-RedHat-9.11.13-2.fc30 <<>> *.extra.df-rosa-1.2wi6.p1.openshiftapps.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5802
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;*.extra.df-rosa-1.2wi6.p1.openshiftapps.com. IN        A

;; ANSWER SECTION:
*.extra.df-rosa-1.2wi6.p1.openshiftapps.com. 60 IN A 18.117.203.26

;; Query time: 37 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Dec 23 12:51:39 EST 2021
;; MSG SIZE  rcvd: 88
```
