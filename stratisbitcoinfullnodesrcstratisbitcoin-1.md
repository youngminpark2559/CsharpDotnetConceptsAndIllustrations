=================================================================================

![](/assets/Stratis.Bitcoin.Builder v1.png)

![](/assets/Stratis.Bitcoin.Builder v1.png)=================================================================================

StratisBitcoinFullNode-src-Stratis.Bitcoin-Broadcasting



StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\BroadcasterBehavior.cs



AttachedNode\_MessageReceivedAsync\(\)

This methods plays a role of a handler for processing incoming message from node.



ProcessMessageAsync\(\)

This method practically handles the following messages which are kind of payload such as TxPayload, MempoolPayload, GetDataPayload, and InvPayload.



ProcessInvPayload\(\)



ProcessGetDataPayload\(\)





---



StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\BroadcasterManagerBase.cs



This class watches if the transaction state changes through the event feature.



And this class has AddOrUpdate\(Transaction transaction, State state\).







---



StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\State.cs





This file includes enumeration type named State, which defines 4 states, CantBroadcast, ToBroadcast, Broadcasted, and Propagated.







---

StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\Success.cs



This file includes enumeration type named Success, which defines 3 states, Yes, DontKnow, and No.







---

StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\TransactionBroadcastEntry.cs



This class is an entry point of the transaction broadcast.

The constructor of this class gets 2 parameters\(Transaction type, State type\).

=================================================================================



