##Cardano Stake Pool Tuning and Experiences

There are many folks within the community helping to try to make the network as solid as it can possibly be, and because I have use\
d some of these efforts, I would like to give credit where it is due:

[PoolTool](pooltool.io)
[ADAPools](adapools.org)
[Failover script](https://github.com/rdlrt/Alternate-Jormungandr-Testnet/blob/master/scripts/jormungandr-leaders-failover.sh)


In addition, here are some recent defects to be aware of, at least during the initial Shelley ITN phase:

[REST API: Passive node promoted to Leader through rest api doesn't see leader logs #1522](https://github.com/input-output-hk/jormu\
ngandr/issues/1522)

[Add support for running multiple jormungandr instances #1468](https://github.com/input-output-hk/jormungandr/issues/1468)

[Node "stops" producing blocks if garbage\_collection\_interval < fragment_ttl #705](https://github.com/input-output-hk/jormungandr\
/issues/705)


###A Tale of Two Hardwares

These are some notes about observations made when tuning two distinct HW platforms to run Cardano's Jormungandr stakepool nodes in \
the Shelley Incentivized Testnet.  

Motivation: Like most in the community, StakeHodlers has access to virtual private servers (VPS) on all of the major platforms, alo\
ng with the skills and knowledge to be able to use VPS infrastructure.  VPS providers (AWS, GCP, etc) provide much convenience.  St\
ill, for a decentralized blockchain project like Cardano, relying too heavily on cloud and PaaS providers takes away some of the de\
centralized nature that we're ultimately trying to achieve.  Everything in due time, and we will strive for good balance in the mea\
ntime.

That said, given that Shelley is still a testnet, StakeHodlers has been experimenting with both cloud nodes and nodes run physicall\
y from a place of business or home.  The network will likely consist of both, and we want to understand some of the pros and cons t\
o make the best choices for our stakepool.

The goal of our stakepool (and probably every other stakepool out there), is nearly 100% uptime.  This must be achieved with the ab\
ility of upgrading the nodes, including the Cardano-specific software and it's underlying OS/runtime environment.  To be able to up\
grade without affecting operations, two basic strategies are available:


