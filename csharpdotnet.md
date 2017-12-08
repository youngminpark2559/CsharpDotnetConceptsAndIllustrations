DotnetAssembly.png![](/assets/DotnetAssembly.png)![](/assets/DotnetAssembly.png)

![](/assets/DotnetAssembly.png)

Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

============================================================================================

DotnetProcessAppdomainThreadContext

![](/assets/DotnetProcessAppdomainThreadContext2.png)o Active threads can be moved to other AppDomains and Context boundaries by the CLR to fit the perfomance.

But it's impossible for a single thread to be working in more than one AppDomain at once.

==============================================================================================

LoadingStoringValueInILcode![](/assets/LoadingStoringValueInILcode.png)================================================================================================

DotnetLanguageCompileProcess![](/assets/DotnetLanguageCompileProcess.png)==================================================================================================

RunDotnetApp![](/assets/RunDotnetApp.png)===============================================================================================

CreateAssemblyByILlanguage

Ref.![](/assets/CreateAssemblyByILlanguage.png)

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

=================================================================================================

DotnetClrCtsClsBaseclass![](/assets/DotnetClrCtsClsBaseclass.png)Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

================================================================================================

DotnetVersionHistory

![](/assets/DotnetVersionHistory.png)

Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

===============================================================================================

ManagedUnmanagedCodeAppAndPlatform

![](/assets/ManagedUnmanagedCodeAppAndPlatform.png)Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

================================================================================================

ThreadAndSynchronous![](/assets/ThreadAndSynchronous.png)Ref.

o [https://codewala.net/2015/07/29/concurrency-vs-multi-threading-vs-asynchronous-programming-explained/](https://codewala.net/2015/07/29/concurrency-vs-multi-threading-vs-asynchronous-programming-explained/)

===============================================================================================

ConcurrencyByMultithreadAndAsynchronous![](/assets/ConcurrencyByMultithreadAndAsynchronous2.png)

===============================================================================================

AsyncAwait![](/assets/AsyncAwait.png)o The thing you should note is that when you use "async and await" feature you're not going to sure whether the task which you're trying to process asynchronously is processed on another thread or same calling thread.

That is decided by the CLR, however, CLR allocates the task which you're trying to process asynchronously on another thread against the calling thread as it can as possible.

==============================================================================================

DelegateAndAsyncButCallingThreadGotBlockedWithWaitingResultFromOtherThread![](/assets/DelegateAndAsyncButCallingThreadGotBlockedWithWaitingResultFromOtherThread.png)Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

============================================================================================

AsyncCallBackInDelegate

![](/assets/AsyncCallBackInDelegate.png)Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

=============================================================================================

LockKeywordForPrivateInstanceMethod![](/assets/LockKeywordForPrivateInstanceMethod.png)

Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

=============================================================================================

LockKeywordForPublicMemberOfTheClass![](/assets/LockKeywordForPublicMemberOfTheClass.png)More techniques:

1.Monitor: This is nothing but a shorthand using for lock keyword.

Using lock keyword is converted to the Monitor using after compiled.

2.Interlock.

3.\[Synchronization\] attribute.

This way can be simple but lazy.

The downfall of this way is that this way makes code rigid because all other codes which doesn't need to be thread-safe are also becoming thread-safe.

4.ThreadPool.

You can pass the method as a argument into WaitCallBack delegate, which you want to execute on secondary threads.

Then delegate object containing above methods can be queued into thread pool via QueueUserWorkItem\(\) method.

And on the secondary threads against primary thread, you can invoke method which you want to invoke.

ThreadPool has pros and cons.

Ref.

o Andrew Troelsen and Philip Japikse - C\# 6.0 and the .NET 4.6 Framework

==========================================================================================

Serialization![](/assets/Serialization.png)==========================================================================================

ObjectGraph1![](/assets/ObjectGraph1.png)==========================================================================================

ObjectGraph2![](/assets/ObjectGraph2.png)==========================================================================================

IFormatterAndIRemotingFormatter

![](/assets/IFormatterAndIRemotingFormatter.png)

==========================================================================================

BinaryAndXmlAndSoapDifferences![](/assets/BinaryAndXmlAndSoapDifferences.png)==========================================================================================

BinaryDataRepresentation![](/assets/BinaryDataRepresentation.png)==========================================================================================

XmlAttributes![](/assets/XmlAttributes.png)==========================================================================================

CustomSerialize![](/assets/CustomSerialize.png)==========================================================================================

chapter13-1 and 13-2

![](/assets/chapter13-1.png)

13-2

![](/assets/chapter13-2.png)

