## Cardano Stake Pool Tuning and Experiences

Author's note: This is very much a preliminary work in progress, and
there is *way* more to come, but the community is attempting to
stabilize the Cardano Shelley testnet, so rather than wait until the
other 80% of this document is ready, I thought I'd release these
initial notes early, to help contribute in some small way by spawning
discussion and spurring community creativity.  All are welcome and
encouraged to contribute to this document (written in Markdown).
While we all want to run successful stakepools with a large number of
stakers, it is more important to get the network running smoothly, and
that should be our primary objective.

There are many folks within the community helping to try to make the
network as solid as it can possibly be, and because I have used some
of these efforts, I would like to give credit where it is due:

- [PoolTool](pooltool.io)

- [ADAPools](adapools.org)

- [Failover script](https://github.com/rdlrt/Alternate-Jormungandr-Testnet/blob/master/scripts/jormungandr-leaders-failover.sh)

- [Jormungandr for Newbs](https://github.com/Chris-Graffagnino/Jormungandr-for-Newbs/blob/master/docs/jormungandr_node_setup_guide.md)

- **_Need the link for ilap's document_**

In addition, here are some recent defects to be aware of, at least during the initial Shelley ITN phase:

[REST API: Passive node promoted to Leader through rest api doesn't see leader logs #1522](https://github.com/input-output-hk/jormungandr/issues/1522)

[Add support for running multiple jormungandr instances #1468](https://github.com/input-output-hk/jormungandr/issues/1468)

[Node "stops" producing blocks if garbage\_collection\_interval < fragment_ttl #705](https://github.com/input-output-hk/jormungandr/issues/705)


### A Tale of Two Hardwares

These are some notes and observations made when tuning two distinct HW
platforms to run Cardano's Jormungandr stakepool nodes in the Shelley
Incentivized Testnet.

Motivation: Like most in the community, StakeHodlers has access to
virtual private servers (VPS) on all of the major platforms, along
with the skills and knowledge to be able to use VPS infrastructure.
VPS providers (AWS, GCP, Digital Ocean, etc) provide much convenience.
Still, for a decentralized blockchain project like Cardano, relying
too heavily on cloud and PaaS providers takes away some of the
decentralized nature that we're ultimately trying to achieve.
Everything in due time, and we will strive for good balance in the
meantime.

That said, given that Shelley is still a testnet, StakeHodlers has
been experimenting with both cloud nodes and nodes run physically from
a place of business or home.  The network will likely consist of both,
and we want to understand some of the pros and cons to make the best
choices for our stakepool.

The goal of our stakepool (and every other stakepool out
there), is nearly 100% uptime.  This must be achieved with the ability
of upgrading the nodes, including the Cardano-specific software and
it's underlying OS/runtime environment.  To be able to upgrade without
affecting operations, two basic strategies are available:

- Incorporate regular maintenance windows into stakepool operations
  (everyone is currently walking around with bags under their eyes
  following this strategy during the nascent days of Shelley);
- The software must provide mechanisms for rolling upgrade of all
  stakepool infrastructure, and (preferably automatic) failover
  capabilities for those inevitable times that either the hardware or
  software craps out.

The stakepool community is doing a good job communicating our needs to
the Cardano development community, and we must keep at it to have the
best possible distributed blockchain ecosystem.  We'll get there!

**Notes and Observations**

1. We noticed that upon reboot, the raspberry pi node would stay in
   sync with the block height reported on pooltool.io much better than
   the Intel node.  After watching a lengthy discussion on Telegram
   about rogue nodes that were creating massive numbers of connections
   to various stakepools, we paid closer attention to active tcp
   connections.  The output of ``` netstat -an | grep ESTABLISHED |
   wc -l``` showed ~750 active connections on the raspberry pi prior
   to shutting down the node and fully rebooting the hardware.  For
   the Intel node, the same command showed ~165 active connections
   prior to shutdown and reboot.  Once the Intel node was restarted,
   it was surprising to see that the active connections remained at
   ~165!  Meanwhile, upon restarting the raspberry pi node, *_ALL_*
   connections were torn down and we started from a single connection
   during the bootstrap phase.  Both nodes are connected to the same
   networking/security infrastructure, with a 1Gb/s pipe out of the
   facility, so the differences could not be attributable to
   differences in ecosystem.  In addition, both nodes are running on
   top of ubuntu server 18.04.3, which takes OS differences out of the
   picture.  This was a strong clue that node instability was at least
   partially attributable to network load, and the interaction of the
   HW on that load.
2. Based upon the first set of observations, along with several in the
   Telegram group discussing the notion of reducing the maximum
   allowable connections, we decided to set the max connections to an
   extremely low value of 32, and the max unresponsive connections
   to 1.  This worked well, but performance (eg: the lead/lag between
   pooltool.io and our pool's block height) degraded over time as the
   number of connections grew.  Clearly this was a call to revisit the
   OS tcp tuning parameters.
3. We've tried both cleaning the SQLite DB directory and leaving it
   alone on reboots.  There is far better and more consistent
   performance by leaving the DB alone during any type of node reset
   event.  If the system gets really wedged, perhaps it is
   occasionally prudent to wipe out the DB and redownload the genesis
   block, but this adds a tremendous amount of time to node-booting.
   Clean the DB only as a last resort.
4. We're using a small list of publicly advertised, trusted peers,
   along with a small collection of passive nodes that are part of the
   stakepool.  Initial bootstrap obviously requires connection to the
   publicly advertised nodes, but the design is such that once the
   passive nodes are relatively stable, restarts can take advantage of
   the reduced latency of connections to the local nodes.
5. I'm reading a discussion on Telegram, regarding the balance between
   supporting the Cardano non-pool-operator community (eg: wallet
   users/stakers) and pool stability.  This balance may be able to be
   achieved by each pool running several passive nodes, but with each
   node allowing fewer external connections.  Using only current
   observations (more observations will be made in the future, and the
   communities thinking and creativity will evolve over time), it
   appears that each jormungandr node has a fairly low tolerance for a
   large number of active tcp connections.  Each node's primary
   responsibility is to maintain the current block height, and produce
   blocks when elected slot leader.  That's it.  At the same time, for
   the stakepools to be useful, they need to share block information
   with the ADA user community.  This suggests that a high-performance
   network architecture would consist of a relatively small number of
   stakepools (~1000?) concentrating on maintaining their
   interconnections with each other, and that smaller network would
   feed into a larger network of passive nodes through a small number
   of channels (eg: passive nodes being part of each pool).  As long
   as the stakepool core doesn't allow external connections to
   bog-down operations, it provides the foundation for everyone else.
   If the stakepool core doesn't adhere to this central tenet, then it
   is diluting itself, and the entire network suffers.
6. There are clearly issues that must be resolved in the jormungandr
   software, and later, the haskell software, in order to make the
   Cardano network the well functioning entity that we know it can be.
   This fact should not detract from the fact that the community needs
   to do it's part in assembling proper devops best practices for
   running the network.  Hopefully many of these best practices will
   be incorporated into the node software over time, but some of it
   will best belong in the hands of the node operators.  Which
   features belong where will be clearer over time, and we must be
   patient to let that process play out naturally.
7. Something definitely worth experimenting with, from Coconutpool on Telegram: "hey, heres something cool you might be interested in. you can use sqlite3 to query the block database directly and you can see what blocks had forks. priyank showed me this its pretty cool. apt install sqlite3 then run this query sqlite3 /path/to/sqlite.blocks "select depth from BlockInfo;" | sort -g its going to give you a long list of all the blocks, when you see double or tripple or more those are forked blocks... The database is full of interesting stuff like the number of fork links (not sure what that means) but lost of the data is serialized so you have to do some work on it to get the real data data out of it"
