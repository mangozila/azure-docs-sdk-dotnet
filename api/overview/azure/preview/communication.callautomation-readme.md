---
title: Azure Communication CallAutomation client library for .NET
keywords: Azure, dotnet, SDK, API, Azure.Communication.CallAutomation, communication
author: acsdevx-msft
ms.author: acsdevx-msft
ms.date: 11/05/2022
ms.topic: reference
ms.devlang: dotnet
ms.service: communication
---
# Azure Communication CallAutomation client library for .NET - version 1.0.0-beta.1 


This package contains a C# SDK for Azure Communication Call Automation.

[Source code][source] | [Product documentation][product_docs]
## Getting started

### Install the package
Install the Azure Communication CallAutomation client library for .NET with [NuGet][nuget]:

```dotnetcli
dotnet add package Azure.Communication.CallAutomation --prerelease
``` 

### Prerequisites
You need an [Azure subscription][azure_sub] and a [Communication Service Resource][communication_resource_docs] to use this package.

To create a new Communication Service, you can use the [Azure Portal][communication_resource_create_portal], the [Azure PowerShell][communication_resource_create_power_shell], or the [.NET management client library][communication_resource_create_net].

### Key concepts
`CallAutomationClient` provides the functionality to answer incoming call or initialize an outbound call.

### Using statements
```C#
using System;
using System.Collections.Generic;
using Azure.Communication.CallAutomation;
```

### Authenticate the client
Call Automation client can be authenticated using the connection string acquired from an Azure Communication Resource in the [Azure Portal][azure_portal].

```C#
var connectionString = "<connection_string>"; // Find your Communication Services resource in the Azure portal
CallAutomationClient callAutomationClient = new CallAutomationClient(connectionString);
```

Or alternatively using a valid Active Directory token.
```C#
var endpoint = new Uri("https://my-resource.communication.azure.com");
TokenCredential tokenCredential = new DefaultAzureCredential();
var client = new CallAutomationClient(endpoint, tokenCredential);
```

## Examples
### Make a call to a phone number recipient
To make an outbound call, call the `CreateCall` or `CreateCallAsync` function from the `CallAutomationClient`.
```C#
CallSource callSource = new CallSource(
       new CommunicationUserIdentifier("<source-identifier>"), // Your Azure Communication Resource Guid Id used to make a Call
       );
callSource.CallerId = new PhoneNumberIdentifier("<caller-id-phonenumber>") // E.164 formatted phone number that's associated to your Azure Communication Resource
```
```C#
CreateCallResult createCallResult = await callAutomationClient.CreateCallAsync(
    source: callSource,
    targets: new List<CommunicationIdentifier>() { new PhoneNumberIdentifier("<targets-phone-number>") }, // E.164 formatted recipient phone number
    callbackEndpoint: new Uri(TestEnvironment.AppCallbackUrl)
    );
Console.WriteLine($"Call connection id: {createCallResult.CallConnectionProperties.CallConnectionId}");
```

### Handle Mid-Connection call back events
Your app will receive mid-connection call back events via the callbackEndpoint you provided. You will need to write event handler controller to receive the events and direct your app flow based on your business logic.
```C#
    /// <summary>
    /// Handle call back events.
    /// </summary>>
    [HttpPost]
    [Route("/CallBackEvent")]
    public IActionResult OnMidConnectionCallBackEvent([FromBody] CloudEvent[] events)
    {
        try
        {
            if (events != null)
            {
                // Helper function to parse CloudEvent to a CallAutomation event.
                CallAutomationEventBase callBackEvent = CallAutomationEventParser.Parse(events.FirstOrDefault());
            
                switch (callBackEvent)
                {
                    case CallConnected ev:
                        # logic to handle a CallConnected event
                        break;
                    case CallDisconnected ev:
                        # logic to handle a CallDisConnected event
                        break;
                    case ParticipantsUpdated ev:
                        # cast the event into a ParticipantUpdated event and do something with it. Eg. iterate through the participants
                        ParticipantsUpdated updatedEvent = (ParticipantsUpdated)ev;
                        break;
                    case AddParticipantsSucceeded ev:
                        # logic to handle an AddParticipantsSucceeded event
                        break;
                    case AddParticipantsFailed ev:
                        # logic to handle an AddParticipantsFailed event
                        break;
                    case CallTransferAccepted ev:
                        # logic to handle CallTransferAccepted event
                        break;
                    case CallTransferFailed ev:
                        # logic to handle CallTransferFailed event
                       break;
                    default:
                        break;
                }
            }
        }
        catch (Exception ex)
        {
            // handle exception
        }
        return Ok();
    }
```
### Idempotent Requests
An operation is idempotent if it can be performed multiple times and have the same result as a single execution.

The following operations are idempotent:
- `AnswerCall`
- `RedirectCall`
- `RejectCall`
- `CreateCall`
- `HangUp` when terminating the call for everyone, ie. `forEveryone` parameter is set to `true`.
- `TransferCallToParticipant`
- `AddParticipants`
- `RemoveParticipants`
- `StartRecording`

By default, SDK generates a new `RepeatabilityHeaders` object every time the above operation is called. If you would
like to provide your own `RepeatabilityHeaders` for your application (eg. for your own retry mechanism), you can do so by specifying
the `RepeatabilityHeaders` in the operation's `Options` object. If this is not set by user, then the SDK will generate
it. You can also disable this by setting `RepeatabilityHeaders` to NULL in the option.

The parameters for the `RepeatabilityHeaders` class are `repeatabilityRequestId` and `repeatabilityFirstSent`. Two or
more requests are considered the same request **if and only if** both repeatability parameters are the same.
- `repeatabilityRequestId`: an opaque string representing a client-generated unique identifier for the request.
It is a version 4 (random) UUID.
- `repeatabilityFirstSent`: The value should be the date and time at which the request was **first** created.

To set repeatability parameters, see below C# code snippet as an example:
```C#
var createCallOptions = new CreateCallOptions(callSource, new CommunicationIdentifier[] { target }, new Uri("https://exmaple.com/callback")) {
    RepeatabilityHeaders = new RepeatabilityHeaders(Guid.NewGuid(), DateTimeOffset.UtcNow);
};
CreateCallResult response1 = await callAutomationClient.CreateCallAsync(createCallOptions).ConfigureAwait(false);
await Task.Delay(5000);
CreateCallResult response2 = await callAutomationClient.CreateCallAsync(createCallOptions).ConfigureAwait(false);
// response1 and response2 will have the same CallConnectionId as they have the same reapeatability parameters which means that the CreateCall operation was only executed once.
```

## Troubleshooting
A `RequestFailedException` is thrown as a service response for any unsuccessful requests. The exception contains information about what response code was returned from the service.

## Next steps
- [Call Automation Overview][overview]
- [Incoming Call Concept][incomingcall]
- [Build a customer interaction workflow using Call Automation][build1]
- [Redirect inbound telephony calls with Call Automation][build2]
- [Quickstart: Play action][build3]
- [Quickstart: Recognize action][build4]
- [Read more about Call Recording in Azure Communication Services][recording1]
- [Record and download calls with Event Grid][recording2]

## Contributing
This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [cla.microsoft.com][cla].

This project has adopted the [Microsoft Open Source Code of Conduct][coc]. For more information see the [Code of Conduct FAQ][coc_faq] or contact [opencode@microsoft.com][coc_contact] with any additional questions or comments.

<!-- LINKS -->
[azure_sub]: https://azure.microsoft.com/free/dotnet/
[azure_portal]: https://portal.azure.com
[cla]: https://cla.microsoft.com
[coc]: https://opensource.microsoft.com/codeofconduct/
[coc_faq]: https://opensource.microsoft.com/codeofconduct/faq/
[coc_contact]: mailto:opencode@microsoft.com
[communication_resource_docs]: /azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp
[communication_resource_create_portal]:  /azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp
[communication_resource_create_power_shell]: /powershell/module/az.communication/new-azcommunicationservice
[communication_resource_create_net]: /azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-net
[product_docs]: /azure/communication-services/overview
[nuget]: https://www.nuget.org/
[source]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.Communication.CallAutomation_1.0.0-beta.1/sdk/communication/Azure.Communication.CallAutomation/src
[overview]: https://learn.microsoft.com/azure/communication-services/concepts/voice-video-calling/call-automation
[incomingcall]: https://learn.microsoft.com/azure/communication-services/concepts/voice-video-calling/incoming-call-notification
[build1]: https://learn.microsoft.com/azure/communication-services/quickstarts/voice-video-calling/callflows-for-customer-interactions?pivots=programming-language-csha
[build2]: https://learn.microsoft.com/azure/communication-services/how-tos/call-automation-sdk/redirect-inbound-telephony-calls?pivots=programming-language-csharp
[build3]: https://learn.microsoft.com/azure/communication-services/quickstarts/voice-video-calling/play-action?pivots=programming-language-csharp
[build4]: https://learn.microsoft.com/azure/communication-services/quickstarts/voice-video-calling/recognize-action?pivots=programming-language-csharp
[recording1]: https://learn.microsoft.com/azure/communication-services/concepts/voice-video-calling/call-recording
[recording2]: https://learn.microsoft.com/azure/communication-services/quickstarts/voice-video-calling/get-started-call-recording?pivots=programming-language-csharp
