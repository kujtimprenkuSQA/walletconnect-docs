import Container from '../../components/Container'
import Tabs from '@theme/Tabs'
import TabItem from '@theme/TabItem'
import PlatformTabs from '../../components/PlatformTabs'
import PlatformTabItem from '../../components/PlatformTabItem'
import CloudBanner from '../../components/CloudBanner'

# Dapp Usage

<CloudBanner/>

## Implementation

<PlatformTabs
groupId="api-sign"
activeOptions={["web","ios","android","flutter"]}>

<PlatformTabItem value="web">
This library is compatible with Node.js, browsers and React Native applications (Node.js modules require polyfills for React Native).

//TODO: [ADD LINKS]

//TODO: Add in links for Node Later.

Dapps will also need to install WalletConnectModal for the UI.

```bash npm2yarn
npm install @walletconnect/modal
```

:::info
For an example implementation, please refer to our `react-dapp-v2` [example](https://github.com/WalletConnect/web-examples/tree/main/dapps/react-dapp-v2).
:::

## Install Packages

Dapps will also need to install `WalletConnectModal` for the UI.

```bash npm2yarn
npm install @walletconnect/modal
```

## Create a Session

**1. Initiate your WalletConnect client with the relay server, using [your Project ID](../../cloud/relay.md).**

```javascript
import SignClient from '@walletconnect/sign-client'

const signClient = await SignClient.init({
  projectId: '<YOUR_PROJECT_ID>',
  // optional parameters
  relayUrl: '<YOUR RELAY URL>',
  metadata: {
    name: 'Example Dapp',
    description: 'Example Dapp',
    url: '#',
    icons: ['https://walletconnect.com/walletconnect-logo.png']
  }
})
```

**2. Add listeners for desired `SignClient` events.**

:::info
To listen to pairing-related events, please follow the guidance for Pairing API event listeners [LINK]
//TODO: Add Pairing to core links.
:::

```javascript
signClient.on('session_event', ({ event }) => {
  // Handle session events, such as "chainChanged", "accountsChanged", etc.
})

signClient.on('session_update', ({ topic, params }) => {
  const { namespaces } = params
  const _session = signClient.session.get(topic)
  // Overwrite the `namespaces` of the existing session with the incoming one.
  const updatedSession = { ..._session, namespaces }
  // Integrate the updated session state into your dapp state.
  onSessionUpdate(updatedSession)
})

signClient.on('session_delete', () => {
  // Session was deleted -> reset the dapp state, clean up from user session, etc.
})
```

**3. Create a new WalletConnectModal instance.**

```javascript
import { WalletConnectModal } from '@walletconnect/modal'

const walletConnectModal = new WalletConnectModal({
  projectId: '<YOUR_PROJECT_ID>',
  // `standaloneChains` can also be specified when calling `walletConnectModal.openModal(...)` later on.
  standaloneChains: ['eip155:1']
})
```

**4. Connect the application and specify session permissions.**

```javascript
try {
  const { uri, approval } = await signClient.connect({
    // Optionally: pass a known prior pairing (e.g. from `signClient.core.pairing.getPairings()`) to skip the `uri` step.
    pairingTopic: pairing?.topic,
    // Provide the namespaces and chains (e.g. `eip155` for EVM-based chains) we want to use in this session.
    requiredNamespaces: {
      eip155: {
        methods: [
          'eth_sendTransaction',
          'eth_signTransaction',
          'eth_sign',
          'personal_sign',
          'eth_signTypedData'
        ],
        chains: ['eip155:1'],
        events: ['chainChanged', 'accountsChanged']
      }
    }
  })

  // Open QRCode modal if a URI was returned (i.e. we're not connecting an existing pairing).
  if (uri) {
    walletConnectModal.openModal({ uri })
    // Await session approval from the wallet.
    const session = await approval()
    // Handle the returned session (e.g. update UI to "connected" state).
    // * You will need to create this function *
    onSessionConnect(session)
    // Close the QRCode modal in case it was open.
    walletConnectModal.closeModal()
  }
} catch (e) {
  console.error(e)
}
```

## Making Requests

Once the session has been established successfully, you can start making JSON-RPC requests to be approved and signed by the wallet:

```javascript
const result = await signClient.request({
  topic: session.topic,
  chainId: 'eip155:1',
  request: {
    method: "personal_sign",
    params: [
      "0x7468697320697320612074657374206d65737361676520746f206265207369676e6564",
      "0x1d85568eEAbad713fBB5293B45ea066e552A90De",
    ],
});
```

> For more information on available JSON-RPC requests, see the [JSON-RPC reference](../../advanced/rpc-reference/ethereum-rpc.md).

## Restoring a Session

Sessions are saved to localstorage, meaning that even if the web page is reloaded, the session can still be retrieved, as demonstrated in the following code:

```ts
const lastKeyIndex = signClient.session.getAll().length - 1
const lastSession = signClient.session.getAll()[lastKeyIndex]
```

## Finding a Specific Session

If you need to find a specific session, you can do so by passing in a known `requiredNamespace` and calling `find`.

```ts
const specificSession = _client.find({
  requiredNamespaces: {
    eip155: {
      methods: [
        'eth_sendTransaction',
        'eth_signTransaction',
        'eth_sign',
        'personal_sign',
        'eth_signTypedData'
      ],
      chains: ['eip155:5'],
      events: ['chainChanged', 'accountsChanged']
    }
  }
})
```

</PlatformTabItem>

<PlatformTabItem value="ios">

### Configure Networking and Pair clients

Make sure that you properly configure Networking and Pair Clients first.

- [Networking] [TODO:ADD LINK]
- [Pairing] [TODO:ADD LINK]

//TODO: Add Core / networking LINKS

//TODO: Add Core / pairing LINKS

### Subscribe for Sign publishers

When your `Sign` instance receives requests from a peer it will publish related event. So you should set subscription to handle them.

To track sessions subscribe to `sessionsPublisher` publisher

```swift
Sign.instance.sessionsPublisher
    .receive(on: DispatchQueue.main)
    .sink { [unowned self] (sessions: [Session]) in
        // reload UI
    }.store(in: &publishers)
```

Following publishers are available to subscribe:

```swift
    public var sessionsPublisher: AnyPublisher<[Session], Never>
    public var sessionProposalPublisher: AnyPublisher<Session.Proposal, Never>
    public var sessionRequestPublisher: AnyPublisher<Request, Never>
    public var socketConnectionStatusPublisher: AnyPublisher<SocketConnectionStatus, Never>
    public var sessionSettlePublisher: AnyPublisher<Session, Never>
    public var sessionDeletePublisher: AnyPublisher<(String, Reason), Never>
    public var sessionResponsePublisher: AnyPublisher<Response, Never>
    public var sessionRejectionPublisher: AnyPublisher<(Session.Proposal, Reason), Never>
    public var sessionUpdatePublisher: AnyPublisher<(sessionTopic: String, namespaces: [String : SessionNamespace]), Never>
    public var sessionEventPublisher: AnyPublisher<(event: Session.Event, sessionTopic: String, chainId: Blockchain?), Never>
    public var sessionUpdateExpiryPublisher: AnyPublisher<(sessionTopic: String, expiry: Date), Never>
```

### Connect Clients

1. Prepare namespaces that constraints minimal requirements for your dApp:

```Swift
let methods: Set<String> = ["eth_sendTransaction", "personal_sign", "eth_signTypedData"]
let blockchains: Set<Blockchain> = [Blockchain("eip155:1")!, Blockchain("eip155:137")!]
let namespaces: [String: ProposalNamespace] = ["eip155": ProposalNamespace(chains: blockchains, methods: methods, events: []]
```

To learn more on namespaces, check out our [specs](../../specs/clients/sign/namespaces).

2. Your App should generate a pairing URI and share it with a wallet. Uri can be presented as a QR code or sent via a universal link. Wallet begins subscribing for session proposals after receiving URI. In order to create a pairing and send a session proposal, you need to call the following:

```Swift
let uri = try await Pair.instance.create()
try await Sign.instance.connect(requiredNamespaces: namespaces, topic: uri.topic)
```

### Send Request to the Wallet

Once the session has been established `sessionSettlePublisher` will publish an event. Your dApp can start requesting wallet now.

```Swift
let method = "personal_sign"
let walletAddress = "0x9b2055d370f73ec7d8a03e965129118dc8f5bf83" // This should match the connected address
let requestParams = AnyCodable(["0x4d7920656d61696c206973206a6f686e40646f652e636f6d202d2031363533333933373535313531", walletAddress])
let request = Request(topic: session.topic, method: method, params: requestParams, chainId: Blockchain(chainId)!)
try await Sign.instance.request(params: request)
```

When wallet respond `sessionResponsePublisher` will publish an event so you can verify the response.

### Where to go from here

- Try our [Example dApp](https://github.com/WalletConnect/WalletConnectSwiftV2/tree/main/Example) that is part of [WalletConnectSwiftV2 repository](https://github.com/WalletConnect/WalletConnectSwiftV2).
- Build API documentation in XCode: go to Product -> Build Documentation

</PlatformTabItem>

<PlatformTabItem value="android">

### **Initialization**

```kotlin
val projectId = "" // Get Project ID at https://cloud.walletconnect.com/
val relayUrl = "relay.walletconnect.com"
val serverUrl = "wss://$relayUrl?projectId=$projectId"
val connectionType = ConnectionType.AUTOMATIC or ConnectionType.MANUAL
val appMetaData = Core.Model.AppMetaData(
    name = "Dapp Name",
    description = "Dapp Description",
    url = "Dapp URL",
    icons = /*list of icon url strings*/,
    redirect = "kotlin-dapp-wc:/request" // Custom Redirect URI
)

CoreClient.initialize(relayServerUrl = serverUrl, connectionType = connectionType, application = this, metaData = appMetaData)

val init = Sign.Params.Init(core = CoreClient)

SignClient.initialize(init) { error ->
    // Error will be thrown if there's an issue during initialization
}
```

The Dapp client is responsible for initiating the connection with wallets and defining the required namespaces (CAIP-2) from the Wallet and is also in charge of sending requests. To initialize the Sign client, create a `Sign.Params.Init` object in the Android Application class with the Core Client. The `Sign.Params.Init` object will then be passed to the `SignClient` initialize function.

#

## **Dapp**

### **SignClient.DappDelegate**

```kotlin
val dappDelegate = object : SignClient.DappDelegate {
    override fun onSessionApproved(approvedSession: Sign.Model.ApprovedSession) {
        // Triggered when Dapp receives the session approval from wallet
    }

    override fun onSessionRejected(rejectedSession: Sign.Model.RejectedSession) {
        // Triggered when Dapp receives the session rejection from wallet
    }

    override fun onSessionUpdate(updatedSession: Sign.Model.UpdatedSession) {
        // Triggered when Dapp receives the session update from wallet
    }

    override fun onSessionExtend(session: Sign.Model.Session) {
        // Triggered when Dapp receives the session extend from wallet
    }

    override fun onSessionEvent(sessionEvent: Sign.Model.SessionEvent) {
        // Triggered when the peer emits events that match the list of events agreed upon session settlement
    }

    override fun onSessionDelete(deletedSession: Sign.Model.DeletedSession) {
        // Triggered when Dapp receives the session delete from wallet
    }

    override fun onSessionRequestResponse(response: Sign.Model.SessionRequestResponse) {
        // Triggered when Dapp receives the session request response from wallet
    }

    override fun onConnectionStateChange(state: Sign.Model.ConnectionState) {
        //Triggered whenever the connection state is changed
    }

    override fun onError(error: Sign.Model.Error) {
        // Triggered whenever there is an issue inside the SDK
    }
}

SignClient.setDappDelegate(dappDelegate)
```

The SignClient needs a `SignClient.DappDelegate` passed to it for it to be able to expose asynchronously updates sent from the Wallet.

#

### **Connect**

```kotlin
val namespace: String = /*Namespace identifier, see for reference: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md#syntax*/
val chains: List<String> = /*List of chains that wallet will be requested for*/
val methods: List<String> = /*List of methods that wallet will be requested for*/
val events: List<String> = /*List of events that wallet will be requested for*/
val requiredNamespaces: Map<String, Sign.Model.Namespaces.Proposal> = mapOf(namespace, Sign.Model.Namespaces.Proposal(accounts, methods, events)) /*Required namespaces to setup a session*/
val optionalNamespaces: Map<String, Sign.Model.Namespaces.Proposal> = mapOf(namespace, Sign.Model.Namespaces.Proposal(accounts, methods, events)) /*Optional namespaces to setup a session*/
val pairing: Core.Model.Pairing = /*Either an active or inactive pairing*/
val connectParams = Sign.Params.Connect(requiredNamespaces, optionalNamespaces, pairing)

fun SignClient.connect(connectParams,
    { onSuccess ->
        /*callback that returns letting you know that you have successfully initiated connecting*/
    },
    { error ->
        /*callback for error while trying to initiate a connection with a peer*/
    }
)
```

More about optional and required namespaces can be found [here](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-25.md)

#

### **Get List of Settled Sessions**

```kotlin
SignClient.getListOfSettledSessions()
```

To get a list of the most current settled sessions, call `SignClient.getListOfSettledSessions()` which will return a list of type `Session`.

#

### **Get list of pending session requests for a topic**

```kotlin
SignClient.getPendingRequests(topic: String)
```

To get a list of pending session requests for a topic, call `SignClient.getPendingRequests()` and pass a topic which will return
a `PendingRequest` object containing requestId, method, chainIs and params for pending request.

</PlatformTabItem>

<PlatformTabItem value="flutter">

### Initialization

To create an instance of `SignClient`, you need to pass in the core and metadata parameters.

```dart
SignClient signClient = await SignClient.createInstance(
    relayUrl: 'wss://relay.walletconnect.com', // The relay websocket URL, leave blank to use the default
    projectId: '123',
    metadata: PairingMetadata(
        name: 'dapp (Requester)',
        description: 'A dapp that can request that transactions be signed',
        url: 'https://walletconnect.com',
        icons: ['https://avatars.githubusercontent.com/u/37784886'],
    ),
);
```

## Connection

To connect with specific parameters and display the returned URI, use `connect` with the required namespaces.

```dart
ConnectResponse response = await signClient.connect(
    requiredNamespaces: {
        'eip155': RequiredNamespace(
            chains: ['eip155:1'], // Ethereum chain
            methods: ['eth_signTransaction'], // Requestable Methods
        ),
        'kadena': RequiredNamespace(
            chains: ['kadena:mainnet01'], // Kadena chain
            methods: ['kadena_quicksign_v1'], // Requestable Methods
        ),
    }
);

Uri? uri = response.uri;
```

You will use that URI to display a QR code or handle a deep link.

We recommend not handling deep linking yourself. If you want to deep link, then use the [walletconnect_modal_flutter](https://pub.dev/packages/walletconnect_modal_flutter) package.

## Session Data

Once you've displayed the URI you can wait for the future and hide the QR code once you've received session data.

```dart
final SessionData session = await response.session.future;
```

## Request Signatures

Once the session had been created, you can request signatures.

```dart
final signature = await signClient.request(
    topic: session.topic,
    chainId: 'eip155:1',
    request: SessionRequestParams(
        method: 'eth_signTransaction',
        params: 'json serializable parameters',
    ),
);
```

## Respond to Events

You can also respond to events from the wallet, like chain changed, using `onSessionEvent` and `registerEventHandler`.

```dart
signClient.onSessionEvent.subscribe((SessionEvent? session) {
    // Do something with the event
});

signClient.registerEventHandler(
    namespace: 'kadena',
    event: 'kadena_transaction_updated',
);
```

# To Test

Run tests using `flutter test`.
Expected flutter version is: >`3.3.10`

# Useful Commands

- `flutter pub run build_runner build --delete-conflicting-outputs` - Regenerates JSON Generators
- `flutter doctor -v` - get paths of everything installed.
- `flutter pub get`
- `flutter upgrade`
- `flutter clean`
- `flutter pub cache clean`
- `flutter pub deps`
- `flutter pub run dependency_validator` - show unused dependencies and more
- `dart format lib/* -l 120`
- `flutter analyze`

</PlatformTabItem>

</PlatformTabs>