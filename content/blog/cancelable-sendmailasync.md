---
title: "Cancelable Sendmailasync"
date: 2018-04-27T20:16:23-07:00
draft: false
---

In .NET, to send emails using the Simple Mail Transfer Protocal we can use the SmtpClient class in System.Net.Mail namespace. There are 3 methods that we can use:

1.  Send
2.  SendAsync
3.  SendMailAsync

The 1st method is a synchronous method. The last 2 methods are asynchronous. I prefer to use async methods most of the time for scalability reason.

SendMailAsync uses SendAsync internally. It also uses [CancellationTokenSource](https://msdn.microsoft.com/en-us/library/system.threading.cancellationtokensource%28v=vs.110%29.aspx) to implement [Task-based Asynchronous Pattern](<https://msdn.microsoft.com/en-us/library/hh873177(v=vs.110).aspx>).

SendAsync operation can be canceled by calling SendAsyncCancel method, so technically you can use that same method to cancel a request made by SendMailAsync. But it would be nicer if SendMailAsync provides an overload that takes a cancellation token. When the token is canceled, SendAsyncCancel would be invoked automatically to cancel the pending request.

This cancelable version of SendMailAsync could be implemented as following

```cs
public class SmtpClientWrapper : IDisposable
{
    private SmtpClient _smtpClient;

    public SmtpClientWrapper(SmtpClient smtpClient)
    {
        this._smtpClient = smtpClient;
    }

    public async Task SendMailAsync(MailMessage message, CancellationToken ct)
    {
        using (ct.Register(_smtpClient.SendAsyncCancel))
        {
            await _smtpClient.SendMailAsync(message);
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool p)
    {
        if (_smtpClient != null)
        {
            _smtpClient.Dispose();
            _smtpClient = null;
        }
    }
}
```

However, there's a problem with this implementation of SendMailAsync. What if the passed in cancellation token has already been canceled before entering the method ?

<!-- more -->

If the token is already in the canceled state, SendAsyncCancel will be run immediately and synchronously. And it will succeed, because there's no pending request. As a result, it will not cancel the actual SendMailAsync request that will be invoked later in the method !!

To make sure we cancel the right request, we need to register the callback _after_ we invoke SendMailAsync.

```csharp
public async Task SendMailAsync(MailMessage message, CancellationToken ct)
{
	try
	{
		var task = _smtpClient.SendMailAsync(message);
		using (ct.Register(_smtpClient.SendAsyncCancel))
		{
			try
			{
				await task;
			}
			catch (OperationCanceledException exception)
			{
				if (exception.CancellationToken == ct)
				{
					Trace.TraceWarning("Operation has been canceled.");
					return;
				}

				throw;
			}
		}
	}
	catch (Exception exception)
	{
		Trace.TraceError("Unexpected exception occured; Exception details: {0}", exception.ToString());
	}
}
```

In this implementation, we start a SendMailAsync first, register the Cancellation callback, then await the SendMailAsync task. If the cancellation token is already in the canceled state before the SendMailAsync method is invoked, we can still cancel it because SendAsyncCancel is registered and invoked after that.
