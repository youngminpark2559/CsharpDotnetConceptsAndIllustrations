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

```
get

{

    lock \(this.lockObject\)

    {

        return this.downloadedBlocks.Count;

    }

}
```

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

---

This AssignDownloadTaskToPeer\(\) assigns a download task to a specific peer.

"peer" represents Peer to be assigned the new task.

"blockHash" represents Hash of the block to download from "peer".

"peerDisconnected" represents if the function fails, this is set to true if the peer was marked as disconnected and thus unable to be assigned any more work.

true: if the block was assigned to the peer.

false: in case the block has already been assigned to someone, or if the peer is disconnected and should not be assigned any more work.

internal bool AssignDownloadTaskToPeer\(BlockPullerBehavior peer, uint256 blockHash, out bool peerDisconnected\){ }

---

This ReleaseDownloadTaskAssignmentLocked\(\) releases the block downloading task from the peer which it has been assigned to and returns the block to the list of blocks which the node wants to download.

"peerPendingDownloads" represents List of pending downloads tasks of the peer.

"blockHash" represents Hash of the block which task should be released.

true: if the function succeeds.

false: if the block was not assigned to be downloaded by any peer.

The caller of this method is responsible for holding "lockObject".

private bool ReleaseDownloadTaskAssignmentLocked\(Dictionary&lt;uint256, DownloadAssignment&gt; peerPendingDownloads, uint256 blockHash\){ }

---

This ReleaseAllPeerDownloadTaskAssignments\(\) releases all pending block download tasks assigned to a peer.

"peer" represents Peer to have all its pending download task released.

"disconnectPeer" represents if set to true, the peer is considered as disconnected and should be prevented from being assigned additional work.

"InvalidOperationException" is thrown in case of data inconsistency between synchronized structures, which should never happen.

internal void ReleaseAllPeerDownloadTaskAssignments\(BlockPullerBehavior peer, bool disconnectPeer\){ }

---

When a peer downloads a block, it notifies the puller about the block by calling this DownloadTaskFinished\(\).

The downloaded task is removed from the list of pending downloads and the downloaded task is also removed from the "assignedBlockTasks" so that the task is no longer assigned to the peer.

And finally, it is added to the list of downloaded blocks, provided that the block is not present there already.

"peer" represents Peer that finished the download task.

"blockHash" represents Hash of the downloaded block.

"downloadedBlock" represents Description of the downloaded block.

true: if the download task for the block was assigned to "peer" and the task was removed and added to the list of downloaded blocks.

false: if the downloaded block has been assigned to another peer

or if the block was already on the list of downloaded blocks.

internal bool DownloadTaskFinished\(BlockPullerBehavior peer, uint256 blockHash, DownloadedBlock downloadedBlock\){ }

---

This GetDownloadedBlock\(\) retrieves a downloaded block from list of downloaded blocks, but does not remove the block from the list.

"blockHash" represents Hash of the block to obtain.

Downloaded block or null if block with the given hash is not on the list.

protected DownloadedBlock GetDownloadedBlock\(uint256 blockHash\){ }

---

This AddDownloadedBlock\(\) adds a downloaded block to the list of downloaded blocks.

If a block with the same hash already existed in the list, it is not replaced with the new one, but the function does not fail.

"blockHash" represents Hash of the block to add.

"downloadedBlock" represents Downloaded block to add.

true: if the block was added to the list of downloaded blocks.

false: if the block was already present.

private bool AddDownloadedBlock\(uint256 blockHash, DownloadedBlock downloadedBlock\){ }

---

This TryRemoveDownloadedBlock\(\) gets and removes a downloaded block from the list of downloaded blocks.

"blockHash" represents Hash of the block to retrieve.

"downloadedBlock" represents if the function succeeds, this is filled with the downloaded block, which hash is "blockHash".

true: if the function succeeds.

false: if the block with the given hash was not in the list.

protected bool TryRemoveDownloadedBlock\(uint256 blockHash, out DownloadedBlock downloadedBlock\){ }

---

This GetPendingDownloadsCount\(\) obtains the number of tasks assigned to a peer.

"peer" represents Peer to get number of assigned tasks for.

Number of tasks assigned to "peer".

internal int GetPendingDownloadsCount\(BlockPullerBehavior peer\){ }

---

This AddPeerPendingDownloadLocked\(\) adds download task to the peer's list of pending download tasks.

"peer" represents Peer to add task to.

"blockHash" represents Hash of the block being assigned to "peer" for download.

The caller of this method is responsible for holding "lockObject".

private void AddPeerPendingDownloadLocked\(BlockPullerBehavior peer, uint256 blockHash\){ }

---

This CheckBlockTaskAssignment\(\) checks if the puller behavior is currently responsible for downloading specific block.

"peer" represents Peer's behavior to check the assignment for.

"blockHash" represents Hash of the block.

true: if the "peer" is currently responsible for downloading block with hash "blockHash".

public bool CheckBlockTaskAssignment\(BlockPullerBehavior peer, uint256 blockHash\){ }

---

\*\*StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\BlockPullerBehavior.cs\*\*

This IBlockPullerBehavior is for relation of the node's puller to a network peer node.

public interface IBlockPullerBehavior{ }

---

public class BlockPullerBehavior : NetworkPeerBehavior, IBlockPullerBehavior{ }

---

This BlockPuller type filed is for Reference to the parent block puller.

private readonly BlockPuller puller;

---

This ChainHeadersBehavior type get set property is for Reference to a component responsible for keeping the chain up to date.

public ChainHeadersBehavior ChainHeadersBehavior { get; private set; }

---

This Disconnected get set property is Set to true: when the puller behavior is disconnected, so that the associated network peer can get no more download tasks.

All access to this object has to be protected by "BlockPuller.lockObject".

internal bool Disconnected { get; set; }

---

This PendingDownloadsCount get property returns the Number of download tasks assigned to this peer. This is for logging purposes only.

public int PendingDownloadsCount { get; }

---

This object type field is for Lock protecting write access to"QualityScore".

private readonly object qualityScoreLock = new object\(\);

---

This double type QualityScore get set property is for Write access to this object which has to be protected by "qualityScoreLock".

public double QualityScore { get; private set; }

---

This constructor BlockPullerBehavior Initializes a new instance of the object with parent block puller.

"puller" represents Reference to the parent block puller.

"loggerFactory" represents Factory to be used to create logger for the puller.

public BlockPullerBehavior\(BlockPuller puller, ILoggerFactory loggerFactory\){ }

---

This Node\_MessageReceived\(\) is Event handler that is called when the attached node receives a network message.

This handler modifies internal state when an information about a block is received.

"node" represents Node that received the message.

"message" represents Received message.

private void Node\_MessageReceived\(NetworkPeer node, IncomingMessage message\){ }

---

If there are any more blocks which the node wants to download, this method assigns and starts

a new download task for a specific peer node that this behavior represents.

internal void AssignPendingVector\(\){ }

---

This StartDownload\(\) Sends a message to the connected peer requesting specific data.

"getDataPayload" represents Specification of the data to download - "GetDataPayload".

Caller is responsible to add the puller to the map if necessary.

internal void StartDownload\(GetDataPayload getDataPayload\){ }

---

This AttachCore\(\) Connects the puller to the node and the chain so that the puller can start its work.

protected override void AttachCore\(\){ }

---

This DetachCore\(\) Disconnects the puller from the node and cancels pending operations and download tasks.

protected override void DetachCore\(\){ }

---

This ReleaseAll\(\) Releases all pending block download tasks from the peer.

"peerDisconnected" represents If set to true, the peer is considered as disconnected and should be prevented from being assigned additional work.

internal void ReleaseAll\(bool peerDisconnected\){ }

---

This UpdateQualityScore\(\) Adjusts the quality score of the peer.

"scoreAdjustment" represents Adjustment to make to the quality score of the peer.

internal void UpdateQualityScore\(double scoreAdjustment\){ }

---

===

\*\*StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\DownloadAssignment.cs\*\*

===

This DownloadAssignment class Describes the assigned download tasks.

public class DownloadAssignment{ }

---

This constructor DownloadAssignment\(\) Initializes a new instance of the object and possibly starts the internal watch.

"blockHash"&gt; represents Hash of the block being downloaded.

"start" represents If true, the download task's stopwatch will be started immediately.

public DownloadAssignment\(uint256 blockHash, bool start = true\){ }

---

This Finish\(\) Stops the task's stopwatch and returns elapsed time.

This Finish\(\) returns Number of milliseconds since the task started.

public long Finish\(\){ }

---

\*\*StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\LookaheadBlockPuller.cs\*\*

---

This ILookaheadBlockPuller is for Block puller used for fast sync during initial block download \(IBD\).

public interface ILookaheadBlockPuller{ }

---

This TryGetLookahead\(\) Tries to retrieve a block that is in front of the current puller's position by a specific height.

"count" represents How many blocks ahead \(minus one\) should the returned block be ahead of the current puller's position.

A value of zero will provide the next block, a value of one will provide a block that is 2 blocks ahead.

This TryGetLookahead\(\) returns The block which height is "count"+1 higher than current puller's position, or null if such a block is not downloaded or does not exist.

Block TryGetLookahead\(int count\);

---

This Location get property Gets the current location of the puller to a specific block header.

ChainedBlock Location { get; }

---

This SetLocation\(\) Sets the current location of the puller to a specific block header.

"location" represents Block header to set the location to.

void SetLocation\(ChainedBlock location\);

---

This NextBlock\(\) Waits for a next block to be available \(downloaded\) and returns it to the consumer.

"cancellationToken" represents Cancellation token to allow the caller to cancel waiting for the next block.

This NextBlock\(\) returns Next block or null if a reorganization happened on the chain.

LookaheadResult NextBlock\(CancellationToken cancellationToken\);

---

This RequestOptions\(\) Adds a specific requirement to all peer nodes.

"transactionOptions" Specifies the requirement on nodes to add.

void RequestOptions\(TransactionOptions transactionOptions\);

---

A result from the "ILookaheadBlockPuller" containing the downloaded block and the "IPEndPoint" of the peer that it came from.

Block will be null if the current chain was reorganized.

public class LookaheadResult{ }

---

This Block type get set property is for The downloaded "Block" that was requested by the puller.

public Block Block { get; set; }

---

This IPEndPoint type get set property is for The peer which this block came from.

public IPEndPoint Peer { get; set; }

---

A puller that is used for fast sync during initial block download \(IBD\). That puller implements a strategy of downloading multiple blocks from multiple peers at once, so that IBD is fast enough, but does not consume too many resources.

The node is aware of the longest chain of block headers, which is stored in this.Chain. This is the chain the puller needs to download. The algorithm works with following values: ActualLookahead, MinimumLookahead, MaximumLookahead, location, and lookaheadLocation.

ActualLookahead is a number of blocks that we wish to download at the same time, it varies between MinimumLookahead and MaximumLookahead depending on the consumer's speed and node download speed. Calling AskBlocks\(\) increases lookaheadLocation by ActualLookahead. Here is a visualization of the block chain and how the puller sees it:

-------A------B-------------C-----------D---------E--------

Each '-' represents a block and letters 'A' to 'E' represent blocks with important positions in the chain from the puller's perspective. The puller can be understood as a producer of the blocks \(by requesting them from the peers and downloading them\) for the component that uses the puller that consumes the blocks \(e.g. validating them\).

A is a position of a block that we call location. Blocks in front of A were already downloaded and consumed.

B is a position of a block A + MinimumLookahead, and E is a position of block A + MaximumLookahead.

Blocks between B and E are the blocks that the puller is currently interested in. Blocks after E are currently not considered and will only be interesting later. The lower boundary B prevents the IBD to be too slow, while the upper boundary E prevents the puller from using too many resources.

Blocks between A and B are blocks that have been downloaded already, but the consumer did not consume them yet.

C is a position of a block A + lookaheadLocation. Blocks between B and C are currently being requested by the puller, some of them could be already being downloaded. The block puller makes sure that if lookaheadLocation &lt; ActualLookahead then AskBlocks\(\) is called. During the initialization, or when reorganisation happens, lookaheadLocation is zero/null and AskBlocks\(\) needs to be called two times.

D is a position of a block A + ActualLookahead. ActualLookahead is a number of blocks that the puller wants to be downloading simultaneously. If there is a gap between C and D it means that the puller wants to start

downloading these blocks.

Blocks between D and E are currently those that the puller does not want to be downloading right now, but should the ActualLookahead be adjusted, they can be requested in the near future.

public class LookaheadBlockPuller : BlockPuller, ILookaheadBlockPuller{ }

---

This int type constant field is for Maximal size of a block in bytes.

private const int MaxBlockSize = 2000000;

---

This int type constant is for Number of milliseconds for a single waiting round for the next block in the "NextBlockCore" loop.

private const int WaitNextBlockRoundTimeMs = 100;

---

This int type get set property is for Lower limit for ActualLookahead.

public int MinimumLookahead { get; set; }

---

This int type get set property is for Upper limit for ActualLookahead.

public int MaximumLookahead { get; set; }

---

This int type field is for Number of blocks the puller wants to be downloading at once.

private int actualLookahead;

---

This int type get set property is for Number of blocks the puller wants to be downloading at once.

public int ActualLookahead { get; set; }

---

This int type get set property is for Maximum number of bytes used by unconsumed blocks that the puller is willing to maintain.

public int MaxBufferedSize { get; set; }

---

This object type field is Lock object to protect access to "currentBufferedSize" ,"currentBufferedCount", and "askBlockQueue".

private readonly object bufferLock = new object\(\);

---

This Queue&lt;ChainedBlock&gt; type field represents Queue of download requests that couldn't be asked for due to "MaxBufferedSize" limit.

All access to this object has to be protected by "bufferLock".

private readonly Queue&lt;ChainedBlock&gt; askBlockQueue = new Queue&lt;ChainedBlock&gt;\(\);

---

This long type field represents Current number of bytes that unconsumed blocks are occupying.

All access to this object has to be protected by "bufferLock".

private long currentBufferedSize;

---

This int type field represents Current number which unconsumed blocks are occupying.

All access to this object has to be protected by "bufferLock".

private int currentBufferedCount;

---

This object type field represents Lock object to protect access to "downloadedCounts".

private readonly object downloadedCountsLock = new object\(\);

---

This List&lt;int&gt; type field Maintains the statistics of number of downloaded blocks. This is used for calculating new actualLookahead value.

All access to this object has to be protected by "downloadedCountsLock".

private List&lt;int&gt; downloadedCounts = new List&lt;int&gt;\(\);

---

This object type field represents Lock object to protect access to "location".

private readonly object locationLock = new object\(\);

---

This ChainedBlock type field represents Points to a block that was consumed last time. The next block returned by the puller to the consumer will be at location + 1.

Write access to this object has to be protected by "locationLock".

private ChainedBlock location;

---

This ChainedBlock type field Identifies the last block that is currently being requested/downloaded.

private ChainedBlock lookaheadLocation;

---

This AutoResetEvent type field represents Event that signals when a downloaded block is consumed.

private readonly AutoResetEvent consumed = new AutoResetEvent\(false\);

---

This AutoResetEvent type field represents Event that signals when a new block is pushed to the list of downloaded blocks.

private readonly AutoResetEvent pushed = new AutoResetEvent\(false\);

---

This decimal type get set property represents Median of a list of past downloadedCounts values. This is used just for logging purposes.

public decimal MedianDownloadCount { get; set; }

---

This constructor Initializes a new instance of the object having a chain of block headers and a connection manager.

"chain" represents Chain of block headers.

"connectionManager" represents Manager of information about the node's network connections.

"loggerFactory" represents Factory to be used to create logger for the puller.

public LookaheadBlockPuller\(ConcurrentChain chain, IConnectionManager connectionManager, ILoggerFactory loggerFactory\)

```
  : base\(chain, connectionManager.ConnectedNodes, connectionManager.NodeSettings.ProtocolVersion, loggerFactory\)
```

{ }

---

This GetMedian\(\) Finds median for list of values.

"sourceNumbers" represents List of values to find median for.

This method returns Median of the input values.

private static decimal GetMedian\(List&lt;int&gt; sourceNumbers\){ }

---

This method Calculates a new value for this.ActualLookahead to keep it within reasonable range.

This ensures that the puller is requesting enough new blocks quickly enough to keep with the demand, but at the same time not too quickly.

private void CalculateLookahead\(\){ }

---

This method Prepares and invokes download tasks from peer nodes for blocks the node is missing.

TODO: Comment is missing here about the details of the logic in this method.

private void AskBlocks\(\){ }

---

This method Adds block download requests to the queue, that later will distribute them to peers.

"downloadRequests" represents Array of block descriptions that need to be downloaded. Must not be empty.

Blocks in the array have to be unique - it is not supported for a single block to be included twice in this array.

private void QueueRequests\(ChainedBlock\[\] downloadRequests\){ }

---

This method Asks for blocks if there is a free space or if the next block is waiting.

Note that this method relies on the fact that the puller requests are ordered.

private void ProcessQueue\(\){ }

---

This method Waits for a next block to be available \(downloaded\).

"cancellationToken" represents Cancellation token to allow the caller to cancel waiting for the next block.

This method returns Next block or null if a reorganization happened on the chain.

private LookaheadResult NextBlockCore\(CancellationToken cancellationToken\){ }

---

---

===

\*\*StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\PullerDownloadAssignments.cs\*\*

===

This class Implements a strategy for a block puller that needs to download a set of blocks from its connected peers.

public static class PullerDownloadAssignments{ }

---

This nested class is for Information about a connected peer.

public class PeerInformation{ }

---

This double type get set property is for Evaluation of the node's past experience with the peer.

public double QualityScore { get; set; }

---

This int type get set property is for Length of the chain which the peer maintains.

public int ChainHeight { get; set; }

---

This object type get set property represents Application defined peer identifier.

public object PeerId { get; set; }

---

This int type get set property represents Number of tasks assigned to this peer.

public int TasksAssignedCount { get; set; }

---

This int type const field is for Number of blocks from the currently last block that are protected from being assigned to poor peers.

"AssignBlocksToPeers"

private const int CriticalLookahead = 10;

---

This int type constant field represents that Peer is considered to be assigned a large amount of work if it has more than this amount of download tasks assigned.

private const int HighWorkAmountThreshold = 50;

---

Having a list of block heights of the blocks that needs to be downloaded and having a list of available peer nodes that can be asked to provide the blocks, this method selects which peer is asked to provide which block.

Past experience with the node is considered when assigning the task to a peer.

Peers with better quality score tend to get more works than others.

"requestedBlockHeights" represents List of block heights that needs to be downloaded.

"availablePeersInformation" represents List of peers that are available including information about lengths of their chains.

This method returns List of block heights that each peer is assigned, mapped by information about peers.

Peers with a lot of work \(more than "HighWorkAmountThreshold"\) which is already assigned to them have less chance of getting more work. However, the quality is stronger factor.

Tasks to download blocks with height in the lower half of the requested block heights are protected from being assigned to peers with quality below the median quality of available peers.

public static Dictionary&lt;PeerInformation, List&lt;int&gt;&gt; AssignBlocksToPeers\(List&lt;int&gt; requestedBlockHeights, List&lt;PeerInformation&gt; availablePeersInformation\){ }

---

This method Chooses random index proportional to the score.

"scores" represents Array of scores.

"totalScore" represents Sum of the values in "scores".

This method returns Random index to "scores" array - i.e. a number from 0 to scores.Length - 1.

private static int GetNodeIndex\(int\[\] scores, int totalScore\){ }

---

\*\*StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\QualityScore.cs\*\*

---

This struct type is for Single historic sample item for quality score calculations.

public struct PeerSample{ }

---

This IBlockPullerBehavior type get set property represents Peer who provided the sample.

public IBlockPullerBehavior peer { get; set; }

---

This double type get set property represents Downloading speed as number of milliseconds per KB.

public double timePerKb { get; set; }

---

This class Implements logic of evaluation of quality of node network peers based on the recent past experience with them with respect to other node's network peers.

Each peer is assigned with a quality score, which is a floating point number between "MinScore" and "MaxScore" inclusive. Each peer starts with the score in the middle of the score interval. The higher the score, the better the peer and the better chance for the peer to get more work assigned.

public class QualityScore{ }

---

This double type constant represents Minimal quality score of a peer node based on the node's past experience with the peer node.

public const double MinScore = 1.0;

---

This doulbe type constatat represents Maximal quality score of a peer node based on the node's past experience with the peer node.

public const double MaxScore = 150.0;

---

This double type get set property represents Average time of a block download among the samples kept in "samples" array.

Write access to this object has to be protected by "lockObject".

Public getter allows better testing of the class.

public double AverageBlockTimePerKb { get; private set; }

---

This CircularArray&lt;PeerSample&gt; type field represents Circular array of recent block times in milliseconds per KB.

All access to this object has to be protected by "lockObject".

private readonly CircularArray&lt;PeerSample&gt; samples;

---

This Dictionary&lt;IBlockPullerBehavior, int&gt; type field represents Reference counter for peers. This is used for calculating how many peers contributed to the sample which history we keep.

All access to this object has to be protected by "lockObject".

private readonly Dictionary&lt;IBlockPullerBehavior, int&gt; peerReferenceCounter;

---

This double type get set property represents Sum of all samples in "samples" array.

All access to this object has to be protected by "lockObject".

private double samplesSum { get; set; }

---

This constructor Initializes a new instance of the object.

"maxSampleCount" represents Maximal number of samples we calculate statistics from.

"loggerFactory" represents Factory to be used to create logger for the puller.

public QualityScore\(int maxSampleCount, ILoggerFactory loggerFactory\){ }

---

&lt;summary&gt;

This method Adds new time of a block to the list of times of recently downloaded blocks.

"peer" represents Peer that downloaded the block.

"blockDownloadTimeMs" represents Time in milliseconds it took to download the block from the peer.

"blockSize" represents Size of the downloaded block in bytes.

public void AddSample\(IBlockPullerBehavior peer, long blockDownloadTimeMs, int blockSize\){ }

---

This method Calculates adjustment of peer's quality score when it finished downloading a block.

"blockDownloadTimeMs" represents Time in milliseconds it took to download the block from the peer.

"blockSize" represents Size of the downloaded block in bytes.

This method returns Quality score adjustment for the peer.

public double CalculateQualityAdjustment\(long blockDownloadTimeMs, int blockSize\){ }

---

This method Calculates peer's penalty when the wait for the next block times out.

This method returns Quality score penalty for the peer.

public double CalculateNextBlockTimeoutQualityPenalty\(\){ }

---

This method Checks whether penalty should be avoided. This is when sum\(score\) of all peers is lower than 2x number of peers peerCount.

This mechanism also prevents single peer to go to minimum if it is alone.

This method returns true if the penalty should be discarded, false otherwise.

public bool IsPenaltyDiscarded\(\){ }

---

===

StratisBitcoinFullNode\src\Stratis.Bitcoin\BlockPulling\StoreBlockPuller.cs

===

---

This class is for Puller that download blocks from peers.

public class StoreBlockPuller : BlockPuller { }

---

This constructor Initializes a new instance of the object having a chain of block headers and a list of available nodes.

"chain" represents Chain of block headers.

"nodes" represents Network peers of the node.

"loggerFactory" represents Factory to be used to create logger for the puller.

public StoreBlockPuller\(ConcurrentChain chain, Connection.IConnectionManager nodes, ILoggerFactory loggerFactory\)

```
        : base\(chain, nodes.ConnectedNodes, nodes.NodeSettings.ProtocolVersion, loggerFactory\)
```

{ }

---

This method Prepares and invokes a download task for multiple blocks.

public void AskForMultipleBlocks\(ChainedBlock\[\] downloadRequests\) { }

---

This method Tries to retrieve a specific downloaded block from the list of downloaded blocks.

"chainedBlock" represents Header of the block to retrieve.

"block" represents If the function succeeds, the downloaded block is returned in this parameter.

This method returns true if the function succeeds, false otherwise.

public bool TryGetBlock\(ChainedBlock chainedBlock, out DownloadedBlock block\){ }

---



