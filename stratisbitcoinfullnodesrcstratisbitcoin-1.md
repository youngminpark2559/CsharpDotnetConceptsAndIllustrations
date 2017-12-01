=================================================================================

![](/assets/Stratis.Bitcoin.Builder v1.png)

![](/assets/Stratis.Bitcoin.Builder v1.png)=================================================================================

**StratisBitcoinFullNode\src\Stratis.Bitcoin\Broadcasting\**

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

**StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\**

StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\BlockPuller.cs

\* This abstract class BlockPuller is base class for pullers who download blocks from peers.

---



\* This BlockPuller class must be inherited \(such as IBlockPuller\) and the implementing class \(such as BlockPuller\) needs to handle taking blocks off the queue and the stalling.



---







\* There are 4 important objects which hold the state of the puller.

And the objects need to be kept in sync.



4 objects:

downloadedBlocks, assignedBlockTasks, pendingInventoryVectors, peersPendingDownloads.





---



\* The downloadedBlocks is a list of blocks which have been downloaded recently but that downloaded block haven't processed yet by the consumer of the puller.





---





\* When a typical consumer wants a next block from the puller, it first checks "downloadedBlocks".





if the block is available \(the consumer does know the header of the block which the consumer wants from the puller. If the consumer doesn't know the header of the block, it simply waits until this information is available.\), the block is removed from "downloadedBlocks" and consumed. 



Otherwise, the consumer checks whether this block is being downloaded \(or soon to be\). 



If not, it asks the puller to request it from the connect network peers.



Besides this "on demand" way of requesting blocks from peers, the consumer also tries to keep puller ahead of the demand, so that the blocks are downloaded some time before they are needed.



For a block to be considered as currently \(or soon to be\) being downloaded, its hash has to be either in "assignedBlockTasks" or "pendingInventoryVectors".





When the puller is about to request blocks from the peers, it selects which of its peers will be asked to provide which blocks. These assignments of block downloading tasks is kept inside "assignedBlockTasks".





Unsatisfied requests go to "pendingInventoryVectors", which happens

when the puller find out that neither of its peers can be asked for certain block. It also happens when something goes wrong \(e.g. the peer disconnects\) and the downloading request to a peer is not completed. Such requests need to be reassigned later. 



Note that it is possible for a peer to be operating well, but slowly, which can cause its quality score to go down and its work to be taken from it. However, this reassignment of the work does not mean the node is stopped in its current task and it is still possible that it will deliver the blocks it was asked for. Such late blocks deliveries are currently ignored and wasted.



---



\* "peersPendingDownloads" is an inverse mapping to "assignedBlockTasks". Each connected peer node which has its list of assigned tasks here and there, is an equivalence between tasks in both structures.











---







\* BlockPuller class includes nested class DownloadedBlock which shows informations such as block size, block itself, the peer which the block came from.





---





This constant means the number of historic samples we keep to calculate quality score stats from.

private const int QualityScoreHistoryLength = 100;



---





This field holding object is used for lock token for preventing the concurrency from being happened by protecting access to "assignedBlockTasks", "pendingInventoryVectors", "downloadedBlocks", "peersPendingDownloads", "peerQuality", and "BlockPullerBehavior.Disconnected".

private readonly object lockObject = new object\(\);



---





This field represents hashes of blocks to be downloaded, mapped by the peers which the download tasks are assigned to.

All access to this object has to be protected by "lockObject".

private readonly Dictionary&lt;uint256, BlockPullerBehavior&gt; assignedBlockTasks;





---





This field represents a list of hash values in the block header, which the node wants to obtain from its peers.

All access to this object has to be protected by "lockObject".

private readonly Queue&lt;uint256&gt; pendingInventoryVectors;







---





This field represent a list of unprocessed downloaded blocks which are mapped by their header hashes.

All access to this object has to be protected by "lockObject".

private readonly Dictionary&lt;uint256, DownloadedBlock&gt; downloadedBlocks;





---





This field represents the statistics of the recent history of network peers qualities.

All access to this object has to be protected by "lockObject".

private readonly QualityScore peerQuality;





---





This get property returns the number of items in "downloadedBlocks". This is for statistical purposes only.

public int DownloadedBlocksCount

{

    get

    {

        lock \(this.lockObject\)

        {

            return this.downloadedBlocks.Count;

        }

    }

}



---



This dictionary type filed represents download tasks representing blocks which are being downloaded, mapped by peers which they are assigned to.

All access to this object has to be protected by "lockObject".



private readonly Dictionary&lt;BlockPullerBehavior, Dictionary&lt;uint256, DownloadAssignment&gt;&gt; peersPendingDownloads = new Dictionary&lt;BlockPullerBehavior, Dictionary&lt;uint256, DownloadAssignment&gt;&gt;\(\);

		

		

		

---



		

This IReadOnlyNetworkPeerCollection type filed represents a collection of available network peers.

protected readonly IReadOnlyNetworkPeerCollection Nodes;







---



This ConcurrentChain type field represents the best chain which the node is aware of.

protected readonly ConcurrentChain Chain;





---



This NetworkPeerRequirement type field represents a specification of requirements which the puller has on its peer nodes to consider asking them to provide blocks.

private readonly NetworkPeerRequirement requirements;







---





This BlockPuller constructor initializes a new instance of the object which is having a chain of block headers and a list of available nodes.



chain represents a chain of block headers.

nodes represents network peers of the node.

protocolVersion represents a version of the protocol which the node supports.

loggerFactory represents a factory to be used to create logger for the puller.



protected BlockPuller\(ConcurrentChain chain, IReadOnlyNetworkPeerCollection nodes, ProtocolVersion protocolVersion, ILoggerFactory loggerFactory\){ }









---







This BlockPushed\(\) is called when a new block is downloaded and pushed to the puller.



This BlockPushed\(\) is to be overridden by derived classes. 

In the base class, it only logs the event.



This method is used to write the logs.



"blockHash" represents a hash value of the newly downloaded block.

"downloadedBlock" represents a desciption of the newly downloaded block.

"cancellationToken" represents a cancellation token to be used by derived classes that allows the caller to cancel the execution of the operation.



public virtual void BlockPushed\(uint256 blockHash, DownloadedBlock downloadedBlock, CancellationToken cancellationToken\){ }







---







This method just invokes BlockPushed\(\) inside of this.



public void InjectBlock\(uint256 blockHash, DownloadedBlock downloadedBlock, CancellationToken cancellationToken\){ }







---





public virtual void AskBlocks\(ChainedBlock\[\] downloadRequests\){ }











---







This method constructs relations to peer nodes which meet the requirements.



Array of relations to peer nodes which can be asked for blocks.

"requirements".



private BlockPullerBehavior\[\] GetNodeBehaviors\(\){ }





---







This method reassigns the incomplete block downloading tasks among available peer nodes.



When something went wrong when the node wanted to download a block from a peer, the task of obtaining the block might get released from the peer. This function leads to assignment of the incomplete tasks to available peer nodes.



private void AssignPendingVectors\(\){ }





---





Manipulate IsDownloading, IsReady where this method caller is, by the value generated inside of this method.

public void CheckBlockStatus\(uint256 hash, out bool IsDownloading, out bool IsReady\){ }







---







This OnStalling\(\) decreases the quality score of the peer node.

This function is called when something goes wrong with the peer.

If the quality score reaches the minimal value, the tasks which are assigned for the node are released.



"chainedBlock"  represents the block which the node wanted to download, but something went wrong during the process.



protected void OnStalling\(ChainedBlock chainedBlock\){ }







---







This DistributeDownload\(\) schedules downloading of one or more blocks, which the node is missing from one or more peer nodes.



Node's quality score is being considered as a weight during the random distribution of the download tasks among the nodes.





Nodes are only asked for blocks that they should have \(according to our information about how long their chains are\).





"vectors" represents list of information about blocks to download, mapped by their height. Must not be empty.

"minHeight" represents Minimum height of the chain that the target nodes has to have in order to be asked for one or more of the block to be downloaded from them.





 private void DistributeDownload\(Dictionary&lt;int, InventoryVector&gt; vectors, int minHeight\){ }

 

 

 



---







This AssignPendingDownloadTaskToPeer\(\) assigns a pending download task to a specific peer.



"peer" represents Peer to be assigned the new task.

"blockHash" represents If the function succeeds, this is filled with the hash of the block that will be requested from "peer".



true: if a download task was assigned to the peer.

false: otherwise, which indicates that there was no pending task, or that the peer is disconnected and should not be assigned any more work.





internal bool AssignPendingDownloadTaskToPeer\(BlockPullerBehavior peer, out uint256 blockHash\){ }



