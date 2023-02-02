# Payments Advanced

While the Pi Frontend Javascript SDK supports creating a U2A (User-To-App) payment, your app might need an ability to
make an A2U (App-To-User) payment. In this documentation, we are going to show you how you can make an A2U payment.

Please note that the A2U payments feature is currently available only on the Testnet.

## Pi Backend SDKs

Here's a list of Pi Backend SDKs for each language (or runtime) you can use for the A2U payment flow.
| Language | Link | Status |
| :---: | :---: | :---: |
| Ruby | [pi-ruby](https://github.com/pi-apps/pi-ruby) | Officially supported by the Pi Core Team |
| Node.js | [pi-nodejs](https://github.com/pi-apps/pi-nodejs) | **Coming soon!** - Officially supported by the Pi Core Team |
| Python | TBD | Community-maintained |
| PHP | TBD | Community-maintained |

Community maintainers for other programming languages are more than welcome, please submit a PR in the [PiOS repository](https://github.com/pi-apps/PiOS) if youare interested to maintain a Pi SDK.


## Supported features

This section is a list of features supported by each backend SDK which enables integrating Pi into an app.
Refer to the specific documentation of the library you're using in your project (see table in the previous section)
for specifics on how to use each feature.

### App-to-user payments

The current design of A2U payments involves both interacting with the Pi Blockchain and the Pi backend. The Pi blockchain is obviously 
the only source of truth for exchanging Pi. The Pi backend is used to improve the end-user experience (e.g. provide users understandable memos 
on their wallet, and link backs to your app, while preserving user and developer privacy by not making this information public on the chain) 
and to assist developers in avoiding to make payment mistakes (e.g. double payments due to server faults)

Benefits:

- **Better user privacy:** This API enables safely accessing a user's wallet address as long as they have consented to
sharing it with your app, and it encourages developers to only access the user's wallet when they have an actual
intent to send them some π, enabling better privacy and user safety.

- **User wallet accuracy:** The API returns the current user wallet if wallet change was necessary, to avoid the app sending Pi to a deprecated,
inaccessible wallet address for the user. 

- **Better experience for users AND app developers:** Once a transaction has happened the blockchain, it can't be reversed.
Letting the Pi backend servers know about your payment before making a transaction, making the transaction and then informing the backend server one 
more time enables recovering the associated metadata and blockchain transaction if a technical issue prevented it from being recorded in your database. 
This way the Pi backend, as a thrid party can help you avoid double payments or other bugs that might accidentally deplete your app wallets by accident.
Of course, there are a lot of things that can go wrong and as the developers of your apps you are the only party responsible for managing your app's 
wallets and assets. The APIs here are provided for as-is and with no guarantees.  

- **Better integration with the Pi ecosystem:** Using the A2U API enables the payment to be shown in the user's wallet as
coming from your app, instead of looking like a random transaction from an unknown address. This can allow beautiful usr experiences in the future such
as direct links backs to your app from the users' wallets or thumbnail icons displaying the goods the users purchased or returned durectly in their wallets. 
