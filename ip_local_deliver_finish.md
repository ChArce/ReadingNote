当通过ip_local_deliver()将分片的packet进行重新组装之后，就会交由ip_local_deliver_finish()函数进行处理。该函数决定了该数据包之后的命运。
它会根据IP头部的协议来决定该交由第四层的哪个handler函数处理，或者进行一些别的操作。
首先我们贴上source code：
192 static int ip_local_deliver_finish(struct net *net, struct sock *sk, struct sk_buff *skb)
193 {
194     __skb_pull(skb, skb_network_header_len(skb));
195
196     rcu_read_lock();
197     {
198         //get the protocol of this packet
199         int protocol = ip_hdr(skb)->protocol;
200         const struct net_protocol *ipprot;// every protocol must register its struct first on init phase
201         int raw;
202
203     resubmit:
204         raw = raw_local_deliver(skb, protocol);
205
206         //from inet_protos list to get the correct protocol struct depending on protocol as index
207         ipprot = rcu_dereference(inet_protos[protocol]);
208         if (ipprot) {
209             int ret;
210
211             //if turn on security policy , deliver to IPSec to process
212             if (!ipprot->no_policy) {
213                 if (!xfrm4_policy_check(NULL, XFRM_POLICY_IN, skb)) {
214                     kfree_skb(skb);
215                     goto out;
216                 }
217                 nf_reset(skb);
218             }
219             //call the Layer 4 handler function,
220             //they are defined in af_inet.c
221             //Here we just trace tcp packets -> call tcp_v4_rcv
222             ret = ipprot->handler(skb); //call the Lay4 handler function
223             if (ret < 0) {
224                 protocol = -ret;
225                 goto resubmit;
226             }
227             __IP_INC_STATS(net, IPSTATS_MIB_INDELIVERS);
228         } else {
229             if (!raw) {
230                 if (xfrm4_policy_check(NULL, XFRM_POLICY_IN, skb)) {
231                     __IP_INC_STATS(net, IPSTATS_MIB_INUNKNOWNPROTOS);
232                     icmp_send(skb, ICMP_DEST_UNREACH,
233                           ICMP_PROT_UNREACH, 0);
234                 }
235                 kfree_skb(skb);
236             } else {
237                 __IP_INC_STATS(net, IPSTATS_MIB_INDELIVERS);
238                 consume_skb(skb);
239             }
240         }
241     }
242  out:
243     rcu_read_unlock();
244
245     return 0;
246 }
首先是更新skb的data指针，现在需要更新使他指向传输层的头部。
